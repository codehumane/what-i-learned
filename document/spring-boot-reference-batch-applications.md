# "How-to" Guides - Batch Applications

[Spring Boot Reference Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/index.html) 문서에 ["How-to" Guides](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto)라고 있음. 그 중에서도 [Batch Applications](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-batch-applications)을 간단히 정리. 아주 짧음.

*스프링 배치 프로젝트 문서는 [여기](https://spring.io/projects/spring-batch) 참고.

## Batch Applications

스프링 부트에서 배치를 사용하면서 종종 생기는 몇 가지 질문들을 다루는 내용.

### Specifying a Batch Data Source

- 배치 애플리케이션은 기본적으로 `DataSource`를 필요로 함.
- 잡 디테일을 저장하기 위함.
- 애플리케이션의 메인 `DataSource` 대신, 배치 전용을 따로 두어 사용하길 권장.
- `@BatchDataSource`와 `@Bean` 애노테이션을 사용.
- 물론 기존의 `DataSource`는 `@Primary`로 두고.
- 좀 더 상세한 제어를 하고 싶다면 `BatchConfigurer`를 구현.
- 자세한 내용은 `@EnableBatchProcessing` [javadoc](https://docs.spring.io/spring-batch/docs/4.2.1.RELEASE/api/org/springframework/batch/core/configuration/annotation/EnableBatchProcessing.html) 참고.

### Running Spring Batch Jobs on Startup

- `@Configuration` 클래스 중에 하나 골라 `@EnableBatchProcessing` 추가
- 이는 스프링 배치 오토 컨피그레이션을 활성화.
- 시작 시 기본적으로 애플리케이션 컨텍스트 안의 모든 `Jobs`를 실행.
- `JobLauncherCommandLineRunner`를 보면 알 수 있음.
- `spring.batch.job.names` 지정으로 범위를 좁힐 수 있음.
- 자세한 내용은 [`BatchAutoConfiguration`](https://github.com/spring-projects/spring-boot/blob/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/BatchAutoConfiguration.java), [`@EnableBatchProcessing`](https://docs.spring.io/spring-batch/docs/4.2.1.RELEASE/api/org/springframework/batch/core/configuration/annotation/EnableBatchProcessing.html) 참고.

### Running from the Command Line

- 스프링 부트의 스프링 배치를 실행시키는 것은 스프링 배치를 실행하는 것과는 다름.
- 가장 주요한 차이는 커맨드 라인 인자를 다르게 파싱.
- 스프링 부트는 `--`로 시작하는 프로퍼티를 모두 `Environment`로 추가함.
- [Accessing Command Line Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config-command-line-args) 참고.
- 배치 잡에 인자를 전달하려면, regular format을 사용. 즉, `--` 사용 안 함.

```sh
$ java -jar myapp.jar someParameter=someValue anotherParameter=anotherValue
```

- 만약 `Environment`의 프로퍼티를 지정한다면, 배치 잡 또한 영향을 받게 됨.
- 아래 예시에서는 배치 잡이 2개의 인자를 받게 됨.
- `someParameter=someValue` 그리고 `-server.port=7070`.
- 스프링 배치는 프로퍼티에서 만약 첫 번째 `-` 문자가 있다면 이를 제거함.

```sh
$ java -jar myapp.jar --server.port=7070 someParameter=someValue
```

### Storing the Job Repository

- `Job` 리포지토리를 위한 데이터 저장소가 필요.
- [Configuring a JobRepository](https://docs.spring.io/spring-batch/docs/4.2.1.RELEASE/reference/html/job.html#configuringJobRepository) 참고.
