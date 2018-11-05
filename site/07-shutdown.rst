==========================
``shutdown``
==========================

``shutdown`` 엔드포인트는 애플리케이션의 정상적인 수행합니다.

``shutdown`` 활성화
=========================

하지만 **종료** 라는 무게만큼 기본적으로 비활성화 되어 있습니다. 먼저 활성화 시켜보겠습니다.

.. code-block:: properties

    management.endpoint.shutdown.enabled=true


* 위 구성을 추가하고 애플리케이션을 재시작합니다.

``shutdown`` 수행
=========================

* http://localhost:8080/actuator/shutdown

하지만 위 페이지로 접근해도 `405` (Method Not Allowed) 오류만 발생합니다.

기본적으로 shutdown 은 행위(동사) 리소스이기 때문에 `POST` 로 호출해야 합니다.

``POST http://localhost:8080/actuator/shutdown``

.. code-block:: bash

    $ http POST localhost:8080/actuator/shutdown
    HTTP/1.1 200
    Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
    Date: Thu, 25 Oct 2018 08:59:50 GMT
    Transfer-Encoding: chunked

    {
        "message": "Shutting down, bye..."
    }


위 응답으로 정상적으로 종료되는 것을 확인할 수 있습니다.


No ``kill -9 {PID}``
=========================

일반적으로 스프링 부트 애플리케이션을 기동할 시에는 `java -jar` 명령어나 docker 또는 클라우드의 PaaS(ex:AWS Elastic Beanstalk)를 많이 사용합니다.

하지만 ``kill`` 과 같은 명령어나 애플리케이션을 종료하는 경우 실행 중인 스레드가 정상적으로 종료되지 않고 강제로 종료되기 때문에 이슈가 발생할 가능성이 있습니다.
그래서 애플리케이션을 종료할 시에는 정상적으로(Gracefully) 종료하는 것이 중요합니다. 이를 위해서 꼭 ``shutdown`` 과 같은 방식을 이용해서 정상적으로 종료할 수 있는 환경을 구축하기를 바랍니다.

:Tips: 전통적인 방식으로 linux에 직접 스프링 부트 애플리케이션을 설치할 시 OS의 서비스로 등록하는 것도 괜찮은 배포 전략입니다.

  * https://docs.spring.io/spring-boot/docs/2.0.6.RELEASE/reference/html/deployment-install.html


``shutdown`` 과 보안
=================================

``shutdown`` 엔드포인트는 보안이 매우 중요합니다. 가능하면 노출하지 않는 것이 좋으며, 만약 노출한다면 꼭 기본적인 보안을 필수로 설정해야합니다.

보안에 관한 세부 내용은 추후 제공될 다음 단계의 Hands-On Labs를 통해서 알아보도록 하며, 여기에서는 간단한 보안 설정만 알아보겠습니다.

*아래 코드 내용은 실습하지 않고 그냥 읽어봅시다*

.. code-block:: bash

    @Configuration
    public class ActuatorSecurity extends WebSecurityConfigurerAdapter {

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.requestMatcher(EndpointRequest.toAnyEndpoint()).authorizeRequests()    // 모든 액추에이터 앤드포인트에
                    .anyRequest().hasRole("ENDPOINT_ADMIN")                             // ENDPOINT_ADMIN 권한일 경우에만 접근 가능
                    .and()
                .httpBasic();
        }
    }
