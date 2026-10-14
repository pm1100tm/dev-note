1. 타임라인 (EKS audit 로그)

- 로그 그룹: /aws/eks/storyroom-prd-stt-eks/cluster
- 먼저 EKS 콘솔 → storyroom-prd-stt-eks → 관찰성 탭 → 컨트롤 플레인 로깅에서 Audit이 켜져 있는지 확인하세요.

fields @timestamp, verb, objectRef.resource as res, objectRef.subresource as sub, objectRef.name as name, user.username as who
| filter @logStream like /^kube-apiserver-audit/ and objectRef.namespace = "storyroom"
| filter (objectRef.resource = "deployments" and objectRef.name = "stt-worker-base"
and objectRef.subresource = "scale" and verb in ["update", "patch"])
or (objectRef.resource = "pods" and objectRef.name like /^stt-worker-base-/
and (objectRef.subresource = "binding" or verb = "delete"))
| sort @timestamp asc

- res=deployments, sub=scale 행에서 who가 system:serviceaccount:keda:keda-operator이면 KEDA가 스케일한 겁니다. audit 로그에는 replica 수가 안 나오므로, 0→1인지 1→0인지는 바로 뒤에 오는 파드 binding/delete 행으로 판단합니다.
- sub=binding 행은 파드가 노드에 배치된(생성된) 시각입니다. verb=delete 행은 종료 시각입니다.

2. 파드가 실제로 떠서 살아 있었는지 (컨테이너 로그)

로그 그룹: /storyroom/stt/prd

fields @timestamp, kubernetes.pod_name as pod
| filter kubernetes.pod_name like /^stt-worker-base-/
| stats min(@timestamp) as first_log, max(@timestamp) as last_log, count(\*) as lines by pod
| sort first_log asc

- 파드 2개가 나와야 합니다. 낮 파드는 last_log가 20시 무렵이고, 밤 파드는 이름이 다르며 first_log가 21시 이후입니다.
- liveness probe 때문에 /health 접근 로그가 20초마다 찍힙니다. 그래서 밤 파드의 last_log가 21:30 무렵이면 그때까지 살아 있었던 겁니다.

3. 처리 가능해진 시각

로그 그룹은 2번과 같은 /storyroom/stt/prd입니다.

fields @timestamp, kubernetes.pod_name as pod, log
| filter kubernetes.pod_name like /^stt-worker-base-/ and log like /STT Worker started/
| sort @timestamp asc

판정할 때 주의할 점

- 20시에 낮 파드가 내려가지 않고 21시까지 이어졌다면 evt1014 덕분이라고 볼 수 없습니다. 이 경우 1번 결과에서 20시 무렵 scale 행과 파드 delete 행이 없고, 2번 결과에서 파드가 하나만 나옵니다. 스케일 다운이 안 된 원인을 따로 봐야 합니다.
- 21:00~21:30 전 구간이 아닐 수 있습니다. cron이 21:00 정각에 시작하는데 노드 배치와 모델 로딩에 10분 정도 걸립니다(매니페스트 주석 기준). 그래서 실제로 처리할 수 있던 구간은 3번 시각부터 21:30까지입니다. 이번 결과가 그렇게 나오면 10/25 이벤트(evt1025, 18:00 시작)도 똑같이 18:10 무렵부터 처리 가능합니다. 정각부터 준비돼 있어야 한다면 biz cron(07:30 시작, 08시 업무 시작)처럼 시작 시각을 10분 이상 앞당기는 게 좋습니다.
- evt1014 설정은 10/13 커밋 e1b71b8에 들어 있습니다. 1번 쿼리에서 21시에 keda-operator의 scale 행이 나오면, 그 설정이 prd에 실제로 배포됐다는 것까지 함께 확인됩니다.

2번과 3번이 비어 있는 이유

로그 수집기(fluent-bit DaemonSet)가 공유 노드에 아예 배포되지 않습니다.

- logging.tf:28: fluent-bit는 nvidia.com/gpu taint만 견딥니다(toleration).
- karpenter.tf:374: base 파드가 뜨는 공유 노드에는 storyroom.io/shared taint가 걸려 있습니다.

그래서 공유 노드에는 fluent-bit가 스케줄되지 않고, 거기서 뜬 base 파드 로그는 CloudWatch에 들어가지 않습니다. 실제로 2번 쿼리의 범위(19:50~)에는 20:00까지 확실히 살아 있던 dgnf7도 포함돼 있었는데 결과가 0건이었습니다. 수집이 안 된다는 증거입니다.

같은 이유로 TTS 노드(storyroom.io/tts)와 음악 노드(storyroom.io/music)에서 뜬 파드 로그도 빠져 있습니다. 그러니 export-base, media-models-base, qwen-tts, music-gen 로그도 지금은 CloudWatch에 없습니다.

4. 컨테이너가 실제로 Ready였는지 확인 (audit 로그)

kubelet이 파드 상태를 바꿀 때 보내는 patch 요청이 audit 로그에 남습니다.

- 로그 그룹: /aws/eks/storyroom-prd-stt-eks/cluster
- 시간 범위: 21:00~21:35

fields @timestamp, user.username as who, @message
| filter @logStream like /^kube-apiserver-audit/
| filter objectRef.resource = "pods" and objectRef.subresource = "status"
and objectRef.name = "stt-worker-base-5ccd65b9f9-zppb6"
| sort @timestamp asc

각 행을 펼쳐서 requestObject.status 안을 보면 됩니다.

- containerStatuses에 "started":true, "ready":true가 있고, Ready 조건이 "status":"True"로 바뀐 시각이 있으면 성공입니다.
  - main.py는 Whisper 모델을 다 올린 뒤에야 HTTP 서버를 엽니다. 그래서 startupProbe의 /health가 통과해 Ready가 됐다면 모델 로딩까지 끝났다는 뜻입니다.
  - Ready로 바뀐 시각이 처리가 가능해진 시점입니다. 아마 21:0x~21:1x일 겁니다.
- 끝까지 waiting(ContainerCreating, ImagePullBackOff 등)이거나 restartCount가 올라가 있으면, 파드는 있었지만 처리할 수 있는 상태가 아니었던 겁니다.
- 결과가 너무 많으면 | filter @message like /"ready":true/를 추가하세요. 첫 행이 Ready가 된 시각입니다.

이번 확인과 별개로, fluent-bit가 모든 노드에 뜨도록 logging.tf의 tolerations를 { operator = "Exists" }로 바꾸면 이런 로그 누락이 해결됩니다. 원하시면 수정안을 만들어 드릴게요. prd에는 terraform apply를 해야 반영됩니다. 10/25 이벤트 전에 반영해 두면 그때는 2·3번 쿼리로 바로 확인할 수 있습니다.

확인됐습니다. stt-worker-base는 evt1014 때문에 21:00:12에 생성됐고, 21:03:53부터 21:30:44까지 약 27분 동안 STT를 처리할 수 있는 상태였습니다. 재시작은 0번이었고, 21:30:44에 정상 종료(exit 0)됐습니다.

타임라인 (KST, 파드 stt-worker-base-5ccd65b9f9-zppb6)

┌──────────┬────────────────────────────────────────────────────┬──────────────────────────────────────────────────────────────────────────────────────┐
│ 시각 │ 상태 │ 의미 │
├──────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
│ 21:00:12 │ 파드 생성, 배치할 노드 없음 (0/2 nodes … didn't │ evt1014로 KEDA가 0→1로 올림. 공유 GPU 노드가 없어서 Karpenter가 새로 띄움 │
│ │ match … selector) │ │
├──────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
│ 21:01:22 │ 노드가 생겼지만 taint 때문에 아직 배치 불가 │ 새 노드가 막 등록되는 중 │
│ │ (untolerated taint) │ │
├──────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
│ 21:01:30 │ Insufficient nvidia.com/gpu │ GPU가 아직 노드에 등록되지 않음 │
├──────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
│ 21:01:43 │ ip-172-31-42-101에 배치, ContainerCreating │ 노드 준비에 약 90초 걸림 │
├──────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
│ 21:03:43 │ 컨테이너 시작 (running, imageID 채워짐) │ 새 노드라 이미지(prd-stt-eeb89a1) 받는 데 약 2분 │
├──────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
│ 21:03:53 │ started=true, ready=true, Ready=True │ 처리 가능 시작. main.py는 모델을 다 올린 뒤에야 HTTP를 열기 때문에, /health가 │
│ │ │ 통과했다면 모델 로딩까지 끝난 것 │
├──────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
│ 21:30:42 │ 삭제 요청 │ evt1014 종료로 KEDA가 0으로 내림 │
├──────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
│ 21:30:44 │ terminated, exitCode 0, Completed, restartCount 0 │ 2초 만에 정상 종료. 종료 시점에 처리 중인 작업은 없었던 것으로 보임 │
└──────────┴────────────────────────────────────────────────────┴──────────────────────────────────────────────────────────────────────────────────────┘

참고할 점

- 21:00:12~21:03:53(약 3분 40초)에는 처리할 수 없었습니다. 원인은 노드 준비 약 90초와 이미지 받기 약 2분입니다. 앞서 매니페스트 주석을 근거로 "약 10분"이라고 했는데, 실제로는 그보다 훨씬 짧았습니다.
