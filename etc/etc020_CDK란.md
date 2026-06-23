# CDK 란?

- CDK는 Cloud Development Kit의 약자입니다.
- AWS에서는 보통 AWS CDK를 의미합니다.
- 간단히 말하면, AWS 인프라를 코드로 작성해서 생성하고 관리할 수 있게 해주는 도구입니다.
- 예를 들어 원래는 AWS 콘솔에서 직접 다음과 같은 리소스를 만들어야 합니다.

```
EC2 생성
VPC 생성
S3 Bucket 생성
RDS 생성
ALB 생성
Route 53 Record 생성
```

그런데 CDK를 사용하면 이런 인프라 구성을 코드로 작성할 수 있습니다.

예를 들어 TypeScript로 S3 Bucket을 만든다면 이런 식입니다.

```ts
import * as cdk from 'aws-cdk-lib';
import { Bucket } from 'aws-cdk-lib/aws-s3';

export class MyInfraStack extends cdk.Stack {
	constructor(scope: cdk.App, id: string) {
		super(scope, id);

		new Bucket(this, 'MyBucket', {
			bucketName: 'my-service-static-files',
		});
	}
}
```

이 코드는 AWS에 S3 버킷을 하나 만드는 인프라 코드입니다.

정리하면 다음과 같습니다.

| 구분      | 설명                                                 |
| --------- | ---------------------------------------------------- |
| CDK       | Cloud Development Kit                                |
| AWS CDK   | AWS 인프라를 코드로 작성하는 도구                    |
| 목적      | AWS 리소스를 코드로 생성, 수정, 삭제                 |
| 사용 언어 | TypeScript, JavaScript, Python, Java, C# 등          |
| 내부 동작 | CDK 코드가 CloudFormation 템플릿으로 변환되어 배포됨 |

즉, AWS CDK는 다음 흐름으로 동작합니다.

```
개발자가 TypeScript/Python 등으로 인프라 코드 작성

        |

        v

CDK가 CloudFormation Template으로 변환

        |

        v

CloudFormation이 AWS 리소스 생성
```

한 줄로 말하면, CDK는 AWS 인프라를 프로그래밍 언어로 관리하는 IaC 도구입니다.

여기서 IaC는 Infrastructure as Code, 즉 인프라를 코드로 관리하는 방식입니다.
