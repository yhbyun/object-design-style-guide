# 3. 다른 객체 생성하기

서비스 객체가 아닌 객체

- 값 객체 (value object)
- entity(model)

## 3.1 일관성 있는 행위에 필요한 최소한의 데이터를 요구한다

코드 3.1 Position 클래스

```php
final class Position
{
    prlvate int x;
    prlvate int y;

    public functlon __construct()
    {
        // 비어있다
    }

    public function setX(int x): void
    {
        this.x = x;
    }

    public function setY(int y): void
    {
        this.y = y;
    }


    public function distanceTo(Position other): float
    {
        return sqrt(
            (other.x - this.x)**2 +
            (other.y - this.y)**2
        );
    }
}

position = new Position();
position.setX(45);
position.setY(60);
```

코드 3.2 Position은 x와 y에 대한 생성자 인자를 요구한다

```php
final class Position
{
    prlvate int x;
    prlvate int y;

    public functlon __construct(int x, int y)
    {
        this.x = x;
        this.y = y;
    }

    public function distanceTo(Position other): float
    {
        return sqrt(
            (other.x - this.x)**2 +
            (other.y - this.y)**2
        );
    }
}

position = new Position(45, 60);
```

이는 생성자를 시용해 `도메인 불변속성(domain invariant)`을 보호하는 방법의 예다. 불변속성이란 주어진 객체가 나타내는 개념에 대한 도메인 지식을 바탕으로 그 객체에 대해 향상 참인 것이다. 여기서 보호한 도메인 불변속성은 '위치에는 x와 y좌표가 모두 있다'이다.

## 3.2 의미 있는 데이터를 요구한다

코드 3.3 Coordinates 클래스

```php
final class Coordinates
{
    prlvate float latitude;
    prlvate float longitude;

    public functlon __construct(flat latitude, float longitude)
    {
        this.latitude = latitude;
        this.longitude = longitude;
    }

    // ...
}

meaningfulCoordinate5 = new Coordinates(45.0, -60.0);

// 말이 안 되는 Coordinates 객처를 민들어도 막을 수 없다.
offThePlanet = newCoordinates(1000.0, -20000.0);
```

클라이언트가 의미 없는 데이터를 제공할 수도 있는지 항상 확인해야 한다. 의미 없는 것으로 간주하는 것은 도메인 불변속성으로 표현할 수도 있다. 이 때 불변속성은 ‘죄표의 위도는 -90에서 90까지 값, 경도는 -180에서 180까지 값이다'가 된다.

코드 3.4 Coordinates 도메인 불변속성 확인하기

```php
expectException(
    InvalidArgumentException.className, // 기대하는 예외 타입
    'Latitude', // 예외 메시지에 있어야 하는 키워드
    function() { // 예외를 일으키는 익명 함수
        new Coordinates(90.1, 0.0);
    }
);

expectException(
    InvalidArgumentException.className,
    'Longitude',
    function() {
        new Coordinates(0.0, 180.1);
    }
);

// 기타 등등...
```

코드 3.5 유효하지 않은 생성자 인자에 대해 예외 일으키기

```php
final class Coordinates
{
    // ...

    public functlon __construct(flat latitude, float longitude)
    {
        if (latitude > 90 || latitude < -90) {
            throw new InvalidArgumentException(
                'Latitude should be between 90 and -90'
            );
        }
        this.latitude = latitude;

        if (longitude > 180 || longitude < -180) {
            throw new InvalidArgumentException(
                'Longitude should be between 180 and -180'
            );
        }
        this.longitude = longitude;
    }
}
```

어떤 때는 모든 생성자 인자 그 자체가 유효한지 확인하는 것으로는 충분치않다. 때로는 생성자  인자가 함께 의미 있는지 확인해야 한다. 다음 예에서는 호텔 예약 정보를 유지하는데 사용하는 ReservationRequest 클래스를 볼 수 있다.

코드 3.6 ReservationRequest 클래스

```php
final class ReservationRequest
{
    public function __construct(
        int numberOfRooms,
        int numberOfAdults,
        int numberOfChildren
    ) {
        // ...
    }
}
```

도메인 전문가와 이 객체에 대한 비즈니스 규칙을 논의하면 다음 규칙을 배울 수 있다

- (어린이는 스스로 방을 예약할 수 없으므로) 최소 어른이 한명 있어야 한다.
- 모두 자신의 방을 가질 수 있지만 손님 수보다 더 많은 방을 예약할 수는 없다 (아무도 자지 않는 방을 예약하게 하는 건 말이 안된다).

코드 3.7 의미 없는 생성자 인자 확인하기

```php
final class ReservationRequest
{

    public function __construct(
        int numberOfRooms,
        int numberOfAdults,
        int numberOfChildren
    ) {
        // 비즈니스 규칙 강제
        if (numberOfRooms > numberOfAdults + numberOfchildren) {
            throw new InvalidArgumentException(
                'Number of rooms should not exceed number of guests'
            );
        }

        if (numberOfAdults < 1) {
            throw new InvalidArgumentException(
                'numberOfAdults should be at least 1'
        );

        if (numberOfChildren < 0) {
            throw new InvalidArgumentException(
                'numberOfChildren should be at least 0'
            );
        );
    }
}

```

첫눈에는 생성자 인자가 연관 있는 것처럼 보이더라도 다시 디자인해 다중 인자 확인을 피할 수도 있다.

코드 3.8 Deal 클래스

```php
final class Deal
{
    public function __construct(
        int totalAmount,
        int amountToFirstParty,
        int amountToSecondParty
    ) {
        // ...
    }
}
```

코드 3.9 Deal에서는 양측의 금액 합을 확인한다

```php
final class Deal
{
    public function __construct(
        int totalAmount,
        int amountToFirstParty,
        int amountToSecondParty
    ) {
        // ...

        if (amountToFirstParty + amountToSecondParty != totalAmount) {
            throw new InvalidArgumentException(
                /* ... */
            );
        }
    }
}
```

코드 3.10 불필요한 생성자 인자 제거하기

```php
final class Deal
{
    public function __construct(
        int amountToFirstParty,
        int amountToSecondParty
    ) {

        if (amountToFirstParty <= 0) {
            throw new InvalidArgumentException(
                /* ... */
            );
        }
        this.amountToFirstParty = amountToFirstParty;

        if (amountToSecondParty <= 0) {
            throw new InvalidArgumentException(
                /* ... */
            );
        }
        this.amountToSecondParty = amountToSecondParty;
    }

    public function totalAmount(): int
    {
        return this.amountToFirstParty + this.amountToSecondParty;
    }
}
```

코드 3.11 Line 클래스

```php
final class Line
{
    public function __construct(
        bool isDotted,
        int distanceBetweenDots
    ) {
        // 선이 점선일 때만 거리를 다룬다. 실선일 때는 처리할 거리가 없다.
        if (isDotted && distanceBetweenDots <= 0) {
            throw new InvalidArgumentException(
                'Expect the distance between dosts to be positive'
            );
        }
    }
}
```

코드 3.12 이제 Line은 생성할 선에 따라 다른 방법을 제공한다

```php
final class Line
{
    private bool isDotted;
    private int distanceBetweenDots;

    // named constructor
    public static function dotted(int distanceBetweenDots): Line
    {
        if (distanceBetweenDots <= 0) {
            throw new InvalidArgumentException(
                'Expect the distance between dosts to be positive'
            );
        }

        line = new Line(/* ... */);
        line.distanceBetweenDots = distanceBetweenDots;
        line.isDotted = true;

        return line;
    }

    // named constructor
    public static function solid(): Line
    {
        line = new Line();
        line.isDotted = false;

        return line;
    }
}
```

## 3.3 유효하지 않은 인자에 대한 예외로 사용자 정의 예외 클래스를 사용하지 않는다

코드 3.13 SpecificException을 붙잡아 처리할 수 있다

```php
final class SpecificException extends InvalidArgumentExCeptlon {
}

try {
    // 객체생성을시도한다
} Catch (SpecificExceptlon exception) {
    // 특정 문제를 특정 방법으로 처리한다
}
```

유효하지 않은 인자는 클라이언트가 유효하지 않은 방법으로 객체를 사용한다는 의미다. 일반적으로 이는 프로그래밍실수로 인한 것이다. 그럴 때는 철저히 실패해, 회복하려 들지 말고 그 실수를 고치는 게 더 낫다.

한편 RuntimeExceptions는 종종 사용자 정의 예외 클래스를 사용하는 게 합당하다. 회복하거나 사용자 친화적인 오류 메시지로 변환할 수 있기 때문이다

## 3.4 예외 메시지를 분석해 유효하지 않은 인자에 대한 특정 예외를 테스트한다

코드 3.15 Coordinates의 도메인 불변속성 데스트

```php
// 위도는 90.0보다 클 수 없다
expectException(
    InvalidArgumentException.className,
    function () {
        new Coordinates(90.1, 0.0);
    }
);

// 위도는 -90.0보다 작을 수 없다
expectException(
    InvalidArgumentException.className,
    function () {
        new Coordinates(-90.1, 0.0);
    }
);

// 경도는 180.0보다 클 수 없다
expectException(
    InvalidArgumentException.className,
    function () {
        new Coordinates(-90.1, 180.1);
    }
);
```

코드 3.16 예외 메시지에 특정 문자열이 있는지 확인하기

```php
expectException(
    InvalidArgumentException.className,
    'Longitude', // 이 단어가 예외 메시지에 있어야 한다
    function () {
        new Coordinates(-90.1, 180.1);
    }
);

```

## 3.5 도메인 불변속성을 여러 곳에서 검증하지 않게 새 객체를 추출한다

코드 3.17 User 클래스

```php
final class User
{
    private string emailAddress;

    public function __construct(string emailAddress)
    {
        // 메일 주소가 유효한지 확인한다
        if (!is_valid_email_address(emailAddress)) {
            throw new InvalidArgumentException(
                'Invalid email address'
            );
        }
        this.emailAddress = emailAddress;
    }

    // ...

    public function changeEmailAddress(string emailAddress): void
    {
        // 이메일 주소를 갱신하면 다시 확인한다
        if (!is_valid_email_address(emailAddress)) {
            throw new InvalidArgumentException(
                'Invalid email address'
            );
        }
        this.emailAddress = emailAddress;
    }
}

// 생성자에서는 유효하지 않은 이메일 주소를 잡아낸다.
expectException(
    InvalidArgumentException.className,
    'email',
    function () {
        new User('not-a-valid-email-address');
    }
);

// 먼저 유효한 User 객체를 생성한다.
user = new User('valid@emailaddress.com');

// changeEmailAddress()에서도 유효하지 않은 이메일 주소를 잡아낸다.
expectException(
    InvalidArgumentException.className,
    'email',
    function () use (User) {
        user.changeEmailAddresS('not-a-valid-email-addre5s');
    }
);
```

코드 3.18 EmailAddress 클래스

```php
final class EmailAddress
{
    private string emailAddress;

    public function __construct(string emailAddress)
    {
        if (!is_valid_email_address(emailAddress)) {
            throw new InvalidArgumentException(
                'Invalid email address'
            );
        }
        this.emailAddress = emailAddress;
    }
}

final class User
{
    private EmailAddress emailAddress;

    public function __construct(EmailAddress emailAddress)
        this.emailAddress = emailAddress;
    }

    // ...

    public function changeEmailAddress(EmailAddress emailAddress): void
    {
        this.emailAddress = emailAddress;
    }
}
```

`값 객체(Value Object)`

메서드가 기본 타입(string, int 등) 값을 받아들인다는 것을 이는 즉시, 이에 대해 클래스를 도입하는 것을 고려해야 한다. 이를 위한 가이드 질문은 'string, int 같은 것이라면 무엇이든 받아들일 수 있는가?'다. 답이 아니라면 해당 개념을 위한 새로운 클래스를 도입한다.

## 3.6 복합 값은 새로운 객체로 추출해 나타낸다

코드 3.19 Amount와 Currency

```php
final class Amount
{
    // ...
}

final class Currency
{
    // ...
}

final class Product
{
    public function setPrice(
        Amount amount,
        Currency currency
    ): void {
        // ...
    }
}

final class Converter
{
    public function convert(
        Amount localAmount,
        Currency localCurrency,
        Currency targetCurrency
    ): Amount {
        // ...
    }
}
```

코드 3.20 Money 클래스

```php
final class Money
{
    public function __construct(Amount amount, Currency currency)
    {
        // ...
    }
}
```

객체 타입을 더 추가할수록 더 많이 타자해야 한다. 그럼에도 정말 필요할끼?

100은 new Amount(100)보다 글자 수가 더 적다. 하지만 객체 타입을 사용하며 더 많이 타자하면 다음과 같은 이점이 있다.

1. 객처로 감싼 데이터의 유효성 확인을 확실히 할 수 있다.
2. 객체는 일반적으로 자신의 데이터를 이용하는 추가적이며 의미 있는 행위를 노출한다.
3. 객체는 함께 속한 값을 함꼐 유지할 수 있다.
4. 객체는 클라이언트가 상세 구현 내용에 접근하지 못하게 하는 데 도움이 된다.

기본 타입 하나하나를 이러한 객처로 생성하는 게 귀찮으면 언제든 이런 객체를 생성하는 도우미 메서드를 도입할 수 있다. 다음 예를 보자.

```php
//이전:
money = new Money(new Amount(100),new Currency('USD'));

// 이후:
money = Money.create(100, 'USD');
```

## 3.7 단언으로 생성자 인자 유효성을 확인한다

```php
if (somethingIsWrong()) {
    throw new InvalidArgumentException(/* ... */);
}
```

메서드 시작 부분에 있는 이러한 확인을 `단언(Assertion)`이라 하며 기본적으로 안전 점검이다. 단언은 싱황을 파악히고 재료를 검토하며 무언가 잘못됐으면 신호를 보내는 데 시용할 수 있다. 이러한 이유로 단언을 ‘사전 조건 확인(precondition check)'이리고도 한다. 일단 이러한 단언을 통과하면 제공받은 데이터로 작업을 수행해도 안전하다.

코드 3.21 EventDispatcher는 생성자에서 단언 함수를 사용한다

```php
final class EventDispatcher
{
    public function __construct(array eventListeners)
    {
        Assertion.allIsInstanceOf(
            eventListeners,
            EventListener.className
        );
        // ...
    }
}
```

코드 3.22 도메인 불변속성에 대한 단위 테스트 추가하기

```php
expectException(
    AssertionFailedException.className,
    'latitude',
    function() {
        new Coordinates(-90.1, 0.0)
    }
);
// and so on...
```

### 예외를 수집하지 않는다

때때로 도구가 허용하더라도 단언 예외를 저장하고 목록으로 전달하지 않아야 한다. 단언은 잘못된 것을 담는 편리한 목록을 사용자에게 제공하기 위한 것이 아니다.  생성자나 메서드를 잘못된 방법으로 사용하는지 알아야 하는 프로그래머를 위한 것이다. 무언가 잘못된 것을 알아내면 바로 객체가 비명을 지르게 한다.

(양식 제출, API 요청 전송 등) 사용자가 제공한 데이터에서 잘못된 것의 목록을 사용자에게 제공하려면 데이터 전송 객체(DTO, data transfer object)를 사용해 유효성을 확인해야 한다.

## 3.8 의존성을 주입하지 말고 선택적인 메서드 인자로 전달한다

서비스는 의존성을 가질 수 있으며 이는 생성자 인자로 주입한다. 하지만 다른 객체에는 값, 값 객체 또는 이런 것의 리스트를 제외하고는 어떤 의존성도 주입하면 안 된다.

코드 3.23 Money는 ExchangeRateProvider 서비스를 필요로 한다

```php
final class Money
{
    private Amount amount;
    private Currency currency;

    public function __construct(Amount amount, Currency currency)
    {
        this.amount = amount;
        this.currency = currency;
    }

    public function convert(
        ExchangeRateProvider exchangeRateProvider,
        Currency targetCurrency
    ): Money {
        exchangeRate = exchangeRateProvider.getRateFor(
            this.currency,
            targetCurrency
        );


        return exchangeRate.convert(this.amount);
    }
}

money = new Money(/* ... */);

converted = money.convert(exchangeRateProvider, targetCurrency);
```

코드 3.24 다른 구현 방법: ExchangeRateProvider를 전달하지 않는다
amount, currency 객체를 드러내야 한다.

```php
final class ExchangeRate
{
    public function __construct(
        Currency from,
        Currenty to,
        Rate rate
    ) {
        // ...
    }

    public function convert(Amount amount): Money
    {
        // ...
    }
}


money = new Money(/* ... */);

exchangeRate = exchangeRateProvider.getRateFor(
    money.currency(),
    targetCurrency
);

converted = exchangeRate.convert(money.amount());
```

코드 3.25 ExchangeRateProvider 대신 ExchangeRate 전달하기
currency 객체만 드러내기

```php
final class Money
{
    private Amount amount;
    private Currency currency;

    public function __construct(Amount amount, Currency currency)
    {
        this.amount = amount;
        this.currency = currency;
    }

    public function convert(ExchangeRate exchangeRate): Money
    {
        Assertion.equals(
            this.currency,
            exchangeRate.fromCurrency()
        )

        return new Money(
            exchangeRate.rate().applyTo(this.amount),
            exchangeRate.targetCurrenty()
        );
    }
}

money = new Money(/* ... */);

exchangeRate = exchangeRateProvider.getRateFor(
    money.currency(),
    targetCurrency
);

converted = money.convert(exchangeRate);
```

이 해결책이 돈과 환율에 대한 도메인 지식을 더 명확히 표현한다고 주장할 수 있다. 예를 들면 환산 금액은 환율의 목표 통화이고 '환산 전(source)' 통화는 원래 금액의 통화와 동일하다.

어떤 사례에서는 서비스를 메서드 인자로 전달해야 한다는 점이 그 행위를 서비스로 구현해야 한다는 암시일 수 있다. 금액을 주어진 통화로 환산하는 사례에서는, 서비스를 생성하고 제공된 Amount와 Currency 객체에서 관련 정보를 모두 수집하여 서비스가 작업을 하도록 할 수도 있다.

코드 3.26 다른 구현 방법: ExchangeService에서 모두 처리한다

```php
final class ExchangeService
{
    private ExchangeRateProvider exchangeRateProvider;

    public function __construct(
        ExchangeRateProvider exchangeRateProvider
    ) {
        this.exchangeRateProvider = exchangeRateProvider;
    }

    public function convert(
        Money money,
        Currency targetCurrency
    ): Money {
        exchangeRate = this.exchangeRateProvider.getRateFor(
            money.currency(),
            targetCurrency
        );


        return new Money(
            exchangeRate.rate().applyTo(money.amount()),
            targetCurrency
        );
    }
}

money = new Money(/* ... */);

exchangeService = new ExchangeService(exchangeRateProvider);

converted = exchangeService.convert(money, targetCurrency);
```

어느 해결책을 선택하느냐는 행위와 데이터를 얼마나 가까이 두기 원히는지, Money와 같은 객체가 환율을 아는 게 너무 과하다 생각하는지, 또는 객체 내부를 노출하는 것을 얼마나 많이 피히고 싶은지에 달렸다.

## 3.9 명명한 생성자를 사용한다

서비스에는 생성자를 정의하는 표준 방법(`public function __construct()`)을 사용하는 게 좋다. 하지만 다른 객체 타입에는 `명명한 생성자(named constructor)`를 사용하길 권한다. 이는 인스턴스를 반환하는 public static 메서드이며, 객체 팩토리로 생각할 수 있다.

### 3.9.1 기본 타입 값으로 생성하기

코드 3.27 Date 클래스는 날짜 문자열을 감싼다.

```php
final class Date
{

    private const string FORMAT = 'd/m/Y';
    private DateTime date;

    // private인 생성자 메서드를 추가해 클라이언트가 명명한 생성자를 우회할 수 없게 하는 게 중요하다.
    private function __construct()
    {
        // do nothing here
    }

    public static function fromString(string date): Date
    {
        object = new Date();

        // 여전히 createFromFormat()이 false를 반환하지 않는 것을 확인해야 한다.
        dateTime = DateTime.createFromFormat(
            Date.FORMAT,
            date
        );

        object.date = dateTime;

        return object;
    }

}

date = Date.fromString('1/4/2019');
```

### 3.9.2 tostring(), tolnt() 등을 즉시 추가하지 말자

fomString() 생성자가 있으면 자동으로 toString() 메서드도 제공하고 싶을 수 있다. 정말 필요할 때만 그렇게 하자.

### 3.9.3 도메인별 개념을 도입한다

코드 3.28 실생활에서는 판매 주문 객체를 생성하는 게 아니라 주문하는 것이다

```php
final class SalesOrder
{
    public static function place(/* ... */): SalesOrder
    {
        // ...
    }
}

salesOrder = SalesOrder.place(/* ... */);
```

### 3.9.4 선택적으로 비공개 생성자를 사용해 제약을 강제한다

코드 3.29 도메인 불변속성을 비공개 생성자 안에 보호하기

```php
final class DecimalValue
{
    private int value;
    private int precision;

    private function __construct(int value, int precision)
    {
        this.value = value;

        Assertion.greaterOrEqualThan(precision, 0);
        this.precision = precision;
    }

    public static function fromInt(
        int value,
        int precision
    ): DecimalValue {
        return new DecimalValue(value, precision);
    }

    public static function fromFloat(
        float value,
        int precision
    ): DecimalValue {
        return new DecimalValue(
            (int)round(value * pow(10, precision)),
            precision
        );
    }

    public static function fromString(string value): DecimalValue
    {
        result = preg_match('/^(\d+)\.(\d+)/', value, matches);
        if (result == 0) {
            throw new InvalidArgumentException(/* ... */);
        }

        wholeNumber = matches[1];
        decimals = matches[2];

        valueWithoutDecimalSign = wholeNumber.decimals;

        return new DecimalValue(
            (int)valueWithoutDecimalSign,
            strlen(decimals)
        );
    }
}
```

명명한 생성자를 사용하면 다음 두 가지 주요 이점을 얻을 수 있다.

- 객체를 생성하는 몇 가지 방법을 제공하는데 사용할 수 있다.
- 객체를 생성하기 위한 도메인별 동의어를 도입하는 데 사용할 수 있다.

## 3.10 속성 채움자를 사용하지 않는다

코드 3.30 Position에는 fromArray() 라는 속성 채움자가 있다

```php
 final class Position
{
    private int x;
    private int y;

    public static function fromArray(array data): Position
    {
        position = new Position();
        position.x = data['x'];
        position.y = data['y'];
        return position;

    }
}
```

이런 메서드는 reflection을 사용해 data 배열에서 그에 상응하는 속성으로 값을 복사하는 일반적인 Utility로 바꿀 수도 있다. 이것이 편리해 보일지라도 객체의 내부가 공개되므로, 객체 생성은 항상 해당  객체에서 완전히 통제하는 방법으로 한다.

## 3.11 무엇이든 필요 이상으로 객체에 넣지 않는다

코드 3.31 ProductCreated 클래스는 이벤트를 나타낸다

```php
final class ProductCreated
{
    public function __construct(
        ProductId productId,
        Description description,
        StockValuation stockValuation,
        Timestamp createdAt,
        UserId createdBy,
        /* ... */
    ){
        // ...
    }
}

this.recordThat(   <-- 제품 개체 안
    new ProductCreated(
        /* ... */  <--- 제품을 생성할 때 사용가능한 모든 데이티를 전달한다
    )
);
```

아직 구현하지 않은 이벤트 수신자에 어떤 이벤트 데이터가 중요한지 모르면 아무것도 추가하지 말자. 인자가 전혀 없는 생성자를 추가했다가, 데이터가 필요할 때 더 많은 데이터를 추가한다. 이런 식으로 알 필요가 있을 때만 데이터를 제공한다.

정말 어떤 데이터를 객체의 생성자에 넣어야 할지 어떻게 알 수 있을끼? 테스트 주도(test-driven) 방법으로 객체를 디자인하면 된다. 이는 곧 객체를 어떻게 사용할지 먼저 알아야 한다는 뜻이다.

## 3.12 생성자는 테스트하지 않는다

코드 3.32 Coordinates의 생성자

```php
final class Coordinates
{
    // ...
    public function __construct(float latitude, float longitude)
    {
        if (latitude > 90 || latitude < -90) {
            throw new InvalidArgumentException(
                'Latitude should be between -90 and 90'
            );
        }
        this.latitude = latitude;

        if (longitude > 180 || longitude < -180) {
            throw new InvalidArgumentException(
                'Longitude should be between -180 and 180'
            );
        }
        this.longitude = longitude;
    }
}
```

코드 3.33 Coordinates의 생성자를 테스트하는 첫 번째 시도

```php
public function it_can_be_constructed(): void
{
    coordinates = new Coordinates(60.0, 100.0);
    assertIsInstanceOf(Coordinates.className, coordinates);
}
```

코드 3.34 Coordinates 생성자를 테스트하는 추가 획득자

```php
final class Coordinates
{
    // ...
    public function latitude(): float
    {
        return this.latitude;
    }

    public function longitude(): float
    {
        return this.longitude;
    }
}
```

코드 3.35 단위 테스트에서 새 획득자 사용하기

```php
public function it_can_be_constructed(): void
{
    coordinates = new Coordinates(60.0, 100.0);
    assertEquals(60.0, coordinates.latitude());
    assertEquals(100.0, coordinates.longitude());
}
```

- 생성자는 실패해야 하는 것만 테스트한다.
- 데이터는 객체의 실제 행위를 구현하는 데 필요할 때만 생성자 인자로 전달한다.
- 내부 데이터를 노출하는 획득자는 그 데이터를 테스트 그 자체가 아닌 다른 클라이언트에서 필요로 할 때만 추가한다

## 3.13 예외 규칙: 데이터 전송 객체

`데이터 전송 객체(DTO)`:   외부에서 들어오는 데이터를 응용 프로그램이 사용할 수 있는 구조로 변환하는 객체

- DTO는 일반적인 생성자로 생성할 수 있다.
- 속성을 하나하나 설정할 수 있다.
- 모든속성을노출한다.
- 속성에는기본 타입 값만 담을 수 있다.
- 속성에는 선택적으로 다른 DTO나 DTO의 단순 배열을 담을 수 있다.

### 3.13.1 공개속성을 사용한다

코드 3.36 ScheduleMeetup DTO

```php
final class ScheduleMeetup
{
    public string title;
    public string date;
}
```

코드 3.37 ScheduleMeetup DTO를 채워 서비스에 전달하기

```php
final class MeetupController
{

    public function scheduleMeetupAction(Request request): Response
    {

        // Extract the form data from the request body.
        formData = /* ... */;

        // Create the command object using this data.
        scheduleMeetup = new ScheduleMeetup();
        scheduleMeetup.title = formData['title'];
        scheduleMeetup.date = formData['date'];

        this.scheduleMeetupService.execute(scheduleMeetup);

        // ...
    }
}
```

### 3.13.2 예외를 일으키지 말고 유효성 오류를 수집한다

코드 3.38 ScheduleMeetup DTO 유효성 확인하기

```php
final class ScheduleMeetup
{

    public string title;
    public string date;

    public function validate(): array
    {
        errors = [];

        if (this.title == '') {
           errors['title'][] = 'validation.empty_title';
        }

        if (this.date == '') {
           errors['date'][] = 'validation.empty_date';
        }

        DateTime.createFromFormat('d/m/Y', this.date);
        errors = DateTime.getLastErrors();
        if (errors['error_count'] > 0) {
            errors['date'][] = 'validation.invalid_date_format';
        }

        return errors;
    }
}
```

### 3.13.3 속성 채움자는 필요할 때 사용한다

코드 3.39 ScheduleMeetup DTO에는 속성 채움자가 있다

```php
final class ScheduleMeetup
{

    public string title;
    public string date;

    public static function fromFormData(
        array formData
    ): ScheduleMeetup {
        scheduleMeetup = new ScheduleMeetup();

        scheduleMeetup.title = formData['title'];
        scheduleMeetup.date = formData['date'];

        return scheduleMeetup;
    }

}
```

## 요약

- 서비스 객체가 아닌 객체는 의존성이 아니라 값이나 값 객체를 받는다. 생성 시점에는 객체가 일관되게 행위를 하는 데 필요한 최소량의 데이터를 제공해야 한다. 생성자 인자가 어떤 식으로든 유효하지 않으면 생성자에서 이에 대한 예외를 일으켜야 한다.
- 기본 타입 인자들 (값) 객체  안으로 감싸는 것은 도움이 된다. 이렇게 하면 이런 값에 대한 유효성 확인 규칙을 재사용하기 쉽다. 또한 그 값의 타입(클래스)에 도메인별 이름을 지정해 코드에 더 많은 의미를 부여하게 된다.
- 서비스가 아닌 객체에서 생성자는 `명명한 생성자`라고 하는 정적메서드여야 한다. 명명한 생성자는 도메인별이름을 코드에 도입하는 또 다른 기회를 제공한다.
- 생성자는 해당 단위 테스트에서 지정한 대로 객체가 행위를 하는 데 필요한 것 이상으로 데이터를 제공하지 않는다.
- 이런 규칙 대부분에 해당히지 않는 객체 타입이 `데이터 전송 객체`다. DTO는 외부에서 제공한 데이터를 나르는 데 사용하며. 내부를 모두 노출한다.
