==========================
주요 웹 엔드포인트
==========================

주요 웹 엔드포인트가 무엇인지 알아보고 한 번씩 엔드포인트에 직접 접근해보자

``beans``
=============================

애플리케이션의 모든 Spring Bean의 전체 목록을 표시

* json을 직접 눈으로 보기에는 너무 정보가 많다.
* http://localhost:8080/actuator/beans

``beans`` 관련 구성

.. code-block:: properties

    management.endpoint.beans.cache.time-to-live=0ms
    management.endpoint.beans.enabled=true


``health``
=============================

애플리케이션의 상태 정보를 표시. **필수적으로 사용하는 엔드포인트**

* 일반적으로 LoadBalancer(ex:L4)에서 해당 애플리케이션 인스턴스의 상태 정보를 확인한다.
* http://localhost:8080/actuator/health

:Note: `6장`_ 에서 자세히 알아보자

.. _6장: 06-health.html


``conditions``
=============================

스프링 부트의 자동설정(``*AutoConfiguration``)과 개발자가 직접 구성한 설정에서 평가된 조건(``@Conditional``)에 관한 정보.
평가가 성공되면 해당 설정이 로딩되고 실패하면 무시한다.

스프링 부트는 자동설정의 평가조건(``@Conditional``)에 따라서 설정이 로딩되거나 무시된다.

* http://localhost:8080/actuator/conditions
* 평가가 성공했으면 ``positiveMatches`` 속성의 항목, 실패했으면 ``negativeMatches`` 속성의 항목이 된다.
* 각 평가에 대한 성공/실패 메시지가 표시된다.

  * 성공 예시 : ``@ConditionalOnEnabledHealthIndicator management.health.defaults.enabled is considered true``
  * 실패 예시 : ``@ConditionalOnClass did not find required class 'com.google.gson.Gson'``


``configprops``
=============================

``@ConfigurationProperties`` 에 대한 정보가 표시된다.

* http://localhost:8080/actuator/configprops

:Tips: ``management.endpoint.configprops.keys-to-sanitize`` 속성을 통해서 민감한 속성은 가릴 수 있다. 예를 들면 비밀번호나 토큰과 같은 정보는 보안 상 노출 시키는 것이 위험하기 때문이다.

.. code-block:: json

    {
        "spring.datasource-org.springframework.boot.autoconfigure.jdbc.DataSourceProperties": {
            "prefix": "spring.datasource",
            "properties": {
                "password": "******",
                "driverClassName": "com.mysql.jdbc.Driver",
                "url": "jdbc:mysql://1.1.1.1:3306/db",
                "username": "username"
            }
        }
    }


``env``
=============================

스프링의 모든 환경변수 정보를 표시한다.

* http://localhost:8080/actuator/env
* 애플리케이션 변수들(``application.properties``) 노출
* OS, JVM 환경변수들 노출

:Tips: ``management.endpoint.env.keys-to-sanitize`` 속성을 통해서 민감한 속성은 가릴 수 있다. 예를 들면 비밀번호나 토큰과 같은 정보는 보안 상 노출 시키는 것이 위험하기 때문이다.

.. code-block:: json

    {
        "spring.datasource.password": {
            "value": "******"
        }
    }



``info``
=============================

임의의 애플리케이션 정보를 표시합니다.

* http://localhost:8080/actuator/info
* 기본적으로 빈 Json Object 반환

:Note: `5장`_ 에서 자세히 알아보자

.. _5장: 05-info.html

``logfile``
=============================

로그 파일의 내용을 반환합니다.

* 현재 웹 애플리케이션 상태에서는 노출되지 않음. 아래 2가지 조건을 만족해야함

  * ``logging.file`` 또는 ``logging.path`` 부트 속성을 이용해서 로그 파일 출력이 활성화

    * 만약 다른 방법으로 로그 파일을 관리한다면 ``management.endpoint.logfile.external-file`` 속성으로 가능
  * 웹 애플리케이션

:Note: 현재 샘플 애플리케이션은 웹 애플리케이션 이긴 하지만 로그파일 출력 설정이 되어 있지 않기 때문에 노출되지 않음.
                    고로 ``application.properties`` 에 ``logging.file`` 속성을 추가해야함

``src/main/resources/application.properties``

.. code-block:: properties

    logging.file=target/application.log

위와 같이 설정하고 애플리 케이션을 재가동 후 아래 엔드포인트에 접근하면 로그를 확인할 수 있다.
추가적으로 `HTTP range requests`_ 를 통해서 로그의 특정 범위만 요청하거나 분할 요청할 수 있다.

.. _`HTTP range requests`: https://developer.mozilla.org/ko/docs/Web/HTTP/Range_requests

* http://localhost:8080/actuator/logfile


``loggers``
=============================

애플리케이션의 Logger 구성을 표시하거나 *수정* 합니다.

* http://localhost:8080/actuator/loggers


``loggers`` 변경
-------------------------

**1. 기본상태**

``GET http://localhost:8080/actuator/loggers/com.nhnent.forward.springbootactuatorlevel1``

.. code-block:: json

    {
        "configuredLevel": null,
        "effectiveLevel": "INFO"
    }



**2. DEBUG로 변경**

.. code-block:: text

    POST http://localhost:8080/actuator/loggers/com.nhnent.forward.springbootactuatorlevel1

    {
        "configuredLevel": "DEBUG"
    }


* ``com.nhnent.forward.springbootactuatorlevel1`` 에 대한 로그 레벨을 ``DEBUG`` 로 변경

**2. DEBUG로 변경 확인**

``GET http://localhost:8080/actuator/loggers/com.nhnent.forward.springbootactuatorlevel1``

.. code-block:: JSON

    {
        "configuredLevel": "DEBUG",
        "effectiveLevel": "DEBUG"
    }


``threaddump``
=============================

스레드 덤프 수행

* http://localhost:8080/actuator/threaddump
* 스레드 덤프 파일을 생성하는 것이 아니라 스레드 덤프 결과를 json 으로 반환


``heapdump``
=============================

GZip으로 압축된 hprof 힙 덤프 파일을 다운로드

* http://localhost:8080/actuator/heapdump
* 웹 애플리케이션 경우에만 사용 가능.

:Warning: Java 애플리케이션에서 **STW(Stop The world)** 가 발생하므로 운영 중인 서비스에서는 사용하지 않는 것이 좋다.

:Tips: `Eclipse Mat`_ 같은 JVM 메모리 분석 도구를 이용해서 해당 내용을 분석할 수 있다.

.. _`Eclipse Mat`: https://www.eclipse.org/mat/


``metrics``
=============================

현재 애플리케이션의 각종 지표(metrics)정보를 표시

* http://localhost:8080/actuator/metrics
* 애플리케이션에 대한 지표 정보를 나열

  * 특정 지표에 대해서 단 건 조회 가능. 아래 `프로세스 CPU 사용률 확인` 참고
* 각종 의존성 라이브러리 추가에 따라서 지표도 추가된다.

  * 만약 DB를 사용한다면 ConnectionPool의 각종 DB 커넥션 수의 정보도 지표로 조회 가능

.. code-block:: json

    {
        "names": [
            "jvm.memory.max",
            "jvm.gc.pause",
            "http.server.requests",
            "process.files.max",
            "jvm.gc.memory.promoted",
            "tomcat.cache.hit",
            "system.load.average.1m",
            "tomcat.cache.access",
            "jvm.memory.used",
            "jvm.gc.max.data.size",
            "jvm.memory.committed",
            "system.cpu.count",
            "process.cpu.usage",
            "#중략"
        ]
    }

프로세스 CPU 사용률 확인
--------------------------

* http://localhost:8080/actuator/metrics/process.cpu.usage

.. code-block:: json

    {
        "name": "process.cpu.usage",
        "description": "The \"recent cpu usage\" for the Java Virtual Machine process",
        "baseUnit": null,
        "measurements": [
            {
                "statistic": "VALUE",
                "value": 0.011448519312787644
            }
        ],
        "availableTags": []
    }


``httptrace``
=============================

최근 100개 HTTP 요청을 반환

* http://localhost:8080/actuator/httptrace
* 응답 모델이 매우 복잡하기 때문에 직접 호출해서 확인해보자

``httptrace`` 관련 구성

.. code-block:: properties

    management.trace.http.include=request-headers,response-headers,cookies,errors

* 노출 시 포함 시킬 Trace 관련 요소들 지정 가능


``mappings``
=============================

모든 ``@RequestMapping`` 경로를 포시

* http://localhost:8080/actuator/mappings


``shutdown``
=============================

애플리케이션을 정상적으로(gracefully) 종료

* ``POST http://localhost:8080/actuator/shutdown``
* GET 명령으로는 실행되지 않는다.

``shutdown`` 관련 기본 구성

.. code-block:: properties

    management.endpoint.shutdown.enabled=false

* 기본적으로 **활성화되어 있지 않음**

:Note: `7장`_ 에서 자세히 알아보자

.. _7장: 07-shutdown.html