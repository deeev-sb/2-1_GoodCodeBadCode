# 1장 잘못된 구조의 문제 깨닫기
## 1.1 의미를 알 수 없는 코드
- 기술 중심 명명(기술을 기반으 이름 붙이는 것), 일련 번호 명명(클래스와 메서드에 번호를 붙여서 이름을 붙이는 것)처럼 기술을 기반으로 이름을 짓거나 번호를 붙여서 이름을 지으면 코드에서 어떤 의도로 이름을 읽어 낼 수 없을 뿐더러 읽고 이해하는데 시간이 오래 걸린다.
- 코드는 계속 변경되는데 문서 유지 보수가 따라가지못하면 로직이 필요 이상으로 복잡해진다.
- 코드는 의도와 목적을 드러내는 이름을 사용하는 것이 좋으며, 이런 경우 구조가 간단하고 명확해진다.


## 1.2 이해하기 어렵게 만드는 조건 분기 중첨
- 조건 분기는 조건에 따라 처리 밥ㅇ식을 다르게 하는 데 사용되는 프로그레밍 언어의 기본 제어 구조이다.
- 조건 분기는 어설프게 하면 이해하고 사용하는 개발자에겐 지옥과 같다.

## 1.3 수많은 악마를 만들어 내는 데이커 클래스
- 데이터 클래스는 설계가 제대로 이루어 지지 않는 SW에서 빈번하게 등장한다.
- 데이터 클래스는 데이터를 갖고만 있는 클래스를 데이터 클래스라고 한다.
    ``` java
    public class Amount {
        public int amount;
        public BigDecimal selesTaxRate;
    }
    ```
- 데이터 클래스는 데이터뿐만 아니라, 금액을 계산하는 로직도 필요한다, 이런 계산을 다른 클래스에 만들어서 구현하는 일이 종종 발생한다. 이런한 구현 방식은 설계를 고려하지 않아 생기는 일이다.
    - 이런 케이스는 작은 규모라면 특별한 문제가 발생하진 않지만, 서비스의 규모가 커진다면 수많은 악마를 부를 수 있다.

    ### 1.3.1 사양을 변경할 때 송곳니를 드러내는 악마
    - 구현 과정에서 요구사항에 따라 특정 기능의 동작이 변경될 수 있다.
    - 하지만 이 동작이 한 곳이 아닌 여러 곳에서 사용되고 있다고 생각했을 때 특정 데이터 클래스에서 선언된 메서드에서 동작하는 구조가 아닌 서로 다른 클래스에서 선언된 메서드를 호출한다고 생각해보자... 이런 경우 요구사항이 변경됨에 따라 동일한 기능을 하는 메서드들을 모두 수정해야한다.
    - 우리는 이것을 응집도가 낮은 구조라고 한다.

    ### 1.3.2 코드 중복
    - 관련된 코드가 서로 멀리 떨어져 있으면, 관련있는 것끼리 묶어서 파악하기 힘들다.

    ### 1.3.3 수정 누락
    - 코드 중복이 많으면, 사양이 변경될 때 중복된 코드를 모두 수정해야한다. 하지만 이 과정에서 일부 코드를 놓치게 되면 해당 기능 대한 버그가 생기게 된다.

    ### 1.3.4 가독성 저하
    - 코드가 분산되어 있으면, 중복된 코드를 포함해서 관련된 정보를 다 찾는 것만으로도 많은 시간이 걸린다. -> 이는 가독성이 많이 떨어진다.
    - 서비스가 작은 경우 문제 없을 수도 있지만, 서비스가 점점 커지게 된다면, 가독성은 정말 중요한 부분이다.

    ### 1.3.5 초기화되지 않는 상태
    ``` java
    ContractAmount amount = new ContractAmount();
    amount.salesTaxRate = new BigDecimal("-0.1");
    ```
    - 이 코드를 실행하면 NPE가 발생한다.
        - `salesTaxRate`는 BigDecimal로 정의되어 있으므로, 따로 초기화를 하지 않는 경우 null이 들어가게 된다.
        - ContractAmount가 추가로 초기화 해야하는 클래스라는 것을 모른다면 버그가 발생하기 쉬운 불안전한 클래스이다.
    - 이처럼 '초기화하지 않으면 쓸모 없는 클래스' 또는 '초기화하지 않은 상태가 발생할 수 있는 클래스'를 안티 패턴인 **쓰레기 객체**라고 부른다.

    ### 1.3.6 잘못된 값 할당
    - 데이터 클래스에서 잘못된 값이 쉽게 들어가는 구조라면 잘못된 값이 들어가지 않게 데이터 클래스를 사용하는 쪽의 로직을 변경하여 유효성 검사가 되도록 해야한다.
    - 하지만 이는 사용하는 곳마다 유효성 검사를 하기에 중복된 코드가 발생 할 수 있어서 조심해야한다.

## 1.4 악마 퇴치의 기본
- 객체 이향의 기본인 클래스는 적절하게 설계 해야한다.


# 2장 설계 첫걸음

## 2.1 의도를 분명히 전달할 수 있는 이름 설계하기
``` java
int d = 0;
d = p1 + p2;
d = d-((d1+d2)) / 2)l
if (d<0) {
    d=0;
}
```
- 위 코드에서 사용하는 변수들은 다른사람이 읽거나 시간이 지나고 다시 보게 되면, 모든 변수들이 어떤 역할을 하는지 알 수 없다.
- 개발은 혼자 하는 것이 아닌 다같이 하는 것이다. 입력하때는 시간을 아끼지만, 해당 코드를 읽고 이해하려는 시간은 2-3배가 될 수 있다.
- 입력하는 시간을 줄이지 말고, 읽고 이해하는 시간을 2-3배 줄이도록 노력하자

## 2.2 목적별로 변수를 따로 만들어 사용하기
- 개념이 다른 값들을 사용할 때는 변수를 재할당 하는 것이 아닌 변수를 따로 만들어서 관리하고 사용하자

## 2.3 단순 나열이 아니라, 의미하는 것을 모아 메서드로 만들기
- 코드를 구현할 때 일련의 흐름이 모두 작성 되어 있으면 로직이 시작점과 끝점을 알기 어렵고 어떤일을 하는지 정확하게 알 수 없다.
- 특히 계산 로직의 경우 게산식이 길어지거나 어려운 경우 많은 실수가 발생 할 수 있다.
- 따라서 코드를 작성 할 때는 단순히 동작하는 순서대로 나열해서 처리하는 것이 아닌 같은 의미를 하는 것을 모아 하나의 메서드로 만들어서 관리해야한다.
    - 이는 코드의 양은 많아졌지만, 코드를 이해하고 읽는데는 훨씬 쉽다.

## 2.4 관련된 데이터와 로직을 클래스로 모으기
- 클래스는 데이터를 인스턴스 변수로 갖고, 인스턴스 변수를 조작하는 메서드를 함께 모아 놓은 것이다.
- 클래스는 의도를 갖고 적절하게 설계하면 유지보수와 변경이 쉬워진다.
