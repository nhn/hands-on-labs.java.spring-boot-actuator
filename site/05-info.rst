==========================
``info``
==========================

기본적으로 ``info`` 엔드포인트는 내용이 없습니다. 개발자가 구성하길 바라는 것입니다.
``info`` 엔드포인트에 내용을 표현하는 방법 2가지(+2)를 알아보겠습니다.

``info.**`` 환경변수
===========================

환경 변수에 ``info.`` 로 시작하는 변수를 추가합니다.
일반적으로 ``application.properties`` 에 추가합니다.

``src/main/resources/application.properties``

.. code-block:: properties

    info.app.name=actuator-levle1
    info.app.version=1.0
    info.app.corporation=nhn

* http://localhost:8080/actuator/info 로 확인해보겠습니다.

.. code-block:: json

    {
        "app": {
            "name": "actuator-levle1",
            "version": "1.0",
            "corporation": "nhn"
        }
    }


빌드 정보
=============

메이븐이나 그레이들 같은 빌드툴의 플러그인을 이용해서 빌드 정보를 노출할 수 있습니다. **개인적으로 추천하는 방법**

메이븐 기준으로 ``pom.xml`` 의 ``spring-boot-maven-plugin`` 부분에 아래와 같은 설정을 추가하면 됩니다.

.. code-block:: xml

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.0.6.RELEASE</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>build-info</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>


:Note: 그레이들은 아래와 같이 구성합니다.

.. code-block:: groovy

    springBoot {
        buildInfo()
    }

이렇게 되면 빌드 산출물 jar 파일 내에 ``META-INF/build-info.properties`` 가 생성되며, 이 정보를 바탕으로 ``info`` 엔드포인트가 표현됩니다.

* http://localhost:8080/actuator/info

:Note: 하지만 IDE 에서 실행할 경우 확인되지 않습니다. IDE 에서 부트 애플리케이션을 실행시키면 메이븐, 그레이들과 같은 툴로 빌드하는 것이 아니라 IDE 자체에서 빌드하기 때문에 ``META-INF/build-info.properties`` 이 생성되지 않습니다.
  아래 과정을 따라서 해보시길 바랍니다.

  1. 터미널 도구를 이용해서 프로젝트 root로 이동.
  2. macOs: ``./mvnw clean package``, windows: ``mvnw.bat clean package``
  3. ``java -jar target/*.jar`` 로 애플리케이션 실행.

    * 이미 기동된 애플리케이션이 있으면 `8080` 포트가 충돌하므로, 기존 애플리케이션은 종료하십시오. 아마도 IDE Run 창에 실행하고 있을 것 입니다.

**결과**

.. code-block:: json

    {
        "build": {
            "version": "0.0.1-SNAPSHOT",
            "artifact": "spring-boot-actuator-level1",
            "name": "spring-boot-actuator-level1",
            "group": "com.nhn.forward",
            "time": "2018-10-25T05:18:50.466Z"
        }
    }

* ``pom.xml`` 의 아래와 같은 정보들이 그대로 노출되는 것을 확인할 수 있습니다.

.. code-block:: xml

    <groupId>com.nhn.forward</groupId>
    <artifactId>spring-boot-actuator-level1</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-boot-actuator-level1</name>


:Note: ``./mvnw``, ``mvnw.bat`` 는 메이븐랩퍼 명령어 입니다.

  * 메이븐 버전 별로 빌드 산출물이 달라질 수 있기 때문에 OS에 설치된 메이븐 버전 의존성에 따른 부수효과(side-effect)를 최소화 하고 일관된 빌드툴 버전을 위해서 `코드베이스` 에 포함되는 툴입니다.
  * 스프링 이니셜라이저(Spring Initializr)를 이용해서 프로젝트를 생성하면 기본적으로 포함됩니다.


그 외 방법 링크
=========================

* `InfoContributor 구현`_
* `Git Commit 정보`_

.. _`InfoContributor 구현`: https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-application-info-custom
.. _`Git Commit 정보`: https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-application-info-git

:Note: ``info.**`` 환경변수, 빌드정보, Git Commit 정보를 합성해서 노출하는 것도 가능합니다.

.. code-block:: json

    {
        "app": {
            "name": "actuator-levle1",
            "version": "1.0",
            "corporation": "nhn"
        },
        "build": {
            "version": "0.0.1-SNAPSHOT",
            "artifact": "spring-boot-actuator-level1",
            "name": "spring-boot-actuator-level1",
            "group": "com.nhn.forward",
            "time": "2018-10-25T05:18:50.466Z"
        },
        "git": {
            "branch": "commit_id_plugin",
            "commit": {
                "id": "7adb64f",
                "time": "2016-08-17T19:30:34+0200"
            }
        }
    }
