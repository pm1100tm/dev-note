# Spring 관련 프로젝트 소개

## Spring Boot

tomcat, netty 등 웹서버를 내장하고 있어 웹 개발하기 편함

## Spring Data

데이터에 접근하기 위한 기술을 모아둔 프로젝트(JDBC, JPA 등)

데이터베이스와 관련된 프로젝트

- JDBC, JPA, Redis, Cassandra, Couchbase, ElasticSearch 등을 지원

## Spring Cloud

분산 시스템에 필요한 다양한 기술을 모아둔 프로젝트(MSA)

클라우드 기반의 어플리케이션을 개발하기 위한 프로젝트

- 분산 시스템에 필요한 기능을 제공
- 라우팅, 부하 분산, 분산 메시징
- MSA

## Spring Security

인증(Authentication)과 인가(Authorization) 관련 기능을 제공하는 프로젝트

- 인증: 사용자가 본인이 맞는지
- 인가: 특정 자원(페이지 등)에 접근 가능한지

## Spring Batch

대용량 처리를 위해 필요한 기술을 모아둔 프로젝트

## 스프링부트의 특징

- 스프링 프레임워크 어플리케이션을 개발할 때 필요한 설정을 모아둠
  - 개발자가 비지니스 개발에 집중할 수 있음
- 기존의 배포 과정 -> 별도 외장 웹 서버 설치 + war 파일로 빌드하여 배포
  - 느리며 번거로움
- 스프링부트는 자체 웹 서버(tomcat, netty)를 내장하고 있음
  - 간편하고 빠르게 배포할 수 있음
