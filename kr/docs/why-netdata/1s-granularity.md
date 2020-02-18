# 초 단위의 높은 데이터 세분성

효과적인 모니터링과 문제 해결 시스템, 응용 프로그램을 위해서는 고 해상도의 측정치가 필요합니다.

## Why?

-   세상는 갈수록 실시간에 가까워지고 있습니다. 최신의 고객 경험은 응답시간에 영향을 크게 받으므로 SLA는 전과 다르게 더 엄격해졌습니다. 이제 10초 단위의 측정치와 2초 단위의 SLA 모니터링은 충분하지 않습니다.

-   IT는 가상화를 향해 가고있습니다. 실제 하드웨어와 다르게 가상 환경은 선형적이지 않고, 예측 가능하지 않습니다. 응용 프로그램이 필요한 만큼의 자원을 사용할 수 있도록 예측하는 것은 힘듭니다. 물론 자원은 요구량 만큼 주어지겠지만 그것들이 필요한 순간 곧바로 주어지지는 않을겁니다. 가상 환경의 지연 시간은 '호스팅 공급 업체의 유지 정책, 자원 할당, 쓰로틀링 정책과 관련 있는 동일한 물리적 서버에서 동작하는 타사의 가상 머신에서 비롯된 작업 부하, 호스팅 공급 업체의 프로비저닝 시스템 등'과 같은 우리가 제어할 수 없는 범위에서 발생하는 여러 요인들로부터 영향을 받습니다. 

## Netdata가 아닌 다른 곳은 어떻습니까?

왜 다른 SaaS 공급 업체나 모니터링 플랫폼은 고해상도의 측정치를 제공하지 않을까요?

그렇게 하고싶지만 하지 못하고 있습니다. 최소한 대규모로는 말입니다.

그 이유는 디자인 결정에 있습니다.:

1.  시계열 데이터베이스 (prometheus, graphite, opentsdb, influxdb 등)은 모든 측정치를 중앙 집중화합니다. 규모에 따라서 이러한 데이터베이스는 전체 인프라에서 병목 현상이 쉽게 발생할 수 있습니다.

2.  SaaS 공급 업체는 모든 측정치를 중앙 집중화하는 비즈니스 모델을 기반으로 합니다. 시분할 데이터베이스의 병목현상 외에도 대역폭 비용이 증가하는 문제 또한 존재합니다. 따라서 고 해상도의 측정치를 대규모로 지원하면 비즈니스 모델이 파괴됩니다.

물론 수십 년이 흐르는 동안 규모에서 발생되는 이런 문제들은 해결되었습니다. 성능을 증가(Scale up)시키는 대신 확장(Scale out)하는 방식으로 말입니다. 즉 매우 큰 중앙 장치에 투자하는 것이 아니라, 애플리케이션을 분산시켜 작은 노드를 더 추가하여 확장할 수 있도록 합니다.

이러한 모니터링의 문제를 해결하기 위한 수많은 시도들이 있었습니다. 하지만 모든 해결책은 측정치를 중앙 집중화해야 했으며 이러한 점은 고성능을 필요로 했습니다. 이러한 이유로 이 문제는 어떻게든 관리되는 것 같지만, 여전히 모든 모니터링 플랫폼의 주요 문제이며 모니터링 비용을 증가시키는 주요 요인 중 하나가 되었습니다.

다른 중요한 요소는 초 단위로 실행시 어떻게 효율적으로 자원 효율적인 데이터 수집이 가능한 지에 대한 것입니다. 대부분의 해결책은 제대로 작동하지 않습니다. 데이터 수집 에이전트는 초 단위로 모니터링할 경우 상당한 시스템 자원을 소비하여 모니터링 시스템과 응용 프로그램에 상당한 영향을 미칩니다.

결론적으로, 초 단위로 데이터를 수집하는 것은 무척 어렵습니다. 바쁘게 동작 중인 가상 환경은 [100ms 일정한 지연 시간을 가지며, 모든 데이터 소스에 무작위로 분산됩니다.](https://docs.google.com/presentation/d/18C8bCTbtgKDWqPa57GXIjB2PbjjpjsUNkLtZEz6YK8s/edit#slide=id.g422e696d87_0_57). 데이터 수집이 제대로 구현되지 않으면 이 지연 시간에 10% 안팎의 무작위 오류가 발생하며, 이는 모니터링 시스템에게 상당히 중요합니다.

정리하자면 모니터링 사업은 주로 3가지의 이유로 대규모의 고 해상도 측정치를 제공하지 않습니다.

1.  측정치를 중앙 집중화하면 모니터링 비용은 비효율적이게 됩니다.
2.  데이터 수집은 최적화가 필요합니다. 그렇지 않으면 모니터링 시스템이 중대한 영향을 미치게 됩니다.
3.  바쁘게 동작 중인 가상 환경에서 특히 더 데이터 수집이 어렵습니다.

## 어떤 점에서 Netdata가 다른가요?

Netdata는 모니터링을 완전히 분산화 시켰습니다. 각각의 Netdata 노드들은 자율적입니다. 로컬에서 측정치를 수집하며 로컬에 저장하고, 경보 발생을 위해 로컬에서 검사하며 대쉬 보드의 시각화를 위해 API를 제공합니다. 이는 Netdata를 무한대로 확장할 수 있게 합니다.

물론 Netdata도 필요에 따라 측정치를 중앙 집중화 할 수 있습니다. 예를 들어 임시 노드에 측정치를 저장하는 것은 실용적이지 않습니다. 이러한 경우 Netdata는 실시간으로 임시 노드에서 근처에 있는 한개 이상의 비임시 노드로 측정치를 스트리밍합니다. 이러한 중앙 집중화되고 나서는 다시 분산화가 이루어집니다. 대규모의 인프라에선 많은 중앙 집중화 노드가 있을 수 있습니다.

바쁘게 동작하는 가상환경에서 데이터 수집 지연 시간으로 발생한 오류를 해결하기 위해서, Netdata는 수집된 측정치를 보간합니다. 데이터 소스 당 마이크로 타이밍을 사용해 0.0001%의 오차율로 측정을 수행합니다. [디버그 모드로 동작 시, Netdata는 오차율 계산](https://github.com/netdata/netdata/blob/36199f449852f8077ea915a3a14a33fa2aff6d85/database/rrdset.c#L1070-L1099)을 수집된 모든 포인트에 대해 수행하여, 데이터베이스가 적절한 정확도에서 동작하도록 합니다.

마지막으로, Netdata는 정말 빠릅니다. 최적화는 핵심적인 기능입니다. 최신 하드웨어에서 Netdata는 각 코어에서 초당 100만 개의 측정치를 수집할 수 있습니다. (여기엔 파싱 데이터 소스나 보간 데이터, 시계열 데이터베이스에 저장된 데이터가 포함됩니다.) 따라서 노드 별 초 당 수 천개의 측정치를 위해서는 Netdata는 무시할만한 CPU 자원을 소비합니다. (싱글 코어에서 1~2% 정도)

Netdata는 다음과 같이 설계되었습니다.

-   모니터링의 중앙 집중화 문제를 해결
-   성능 문제 해결을 위한 콘솔을 대체

따라서 Netdata에게 1초 단위의 데이터 세분성을 갖는 것은 쉽고 자연스러운 결과입니다...

[![analytics](https://www.google-analytics.com/collect?v=1&aip=1&t=pageview&_s=1&ds=github&dr=https%3A%2F%2Fgithub.com%2Fnetdata%2Fnetdata&dl=https%3A%2F%2Fmy-netdata.io%2Fgithub%2Fdocs%2Fwhy-netdata%2F1s-granularity&_u=MAC~&cid=5792dfd7-8dc4-476b-af31-da2fdb9f93d2&tid=UA-64295674-3)](<>)