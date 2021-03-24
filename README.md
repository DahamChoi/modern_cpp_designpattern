
# modern_cpp_designpattern 
모던 C++ 디자인 패턴(저 : 드미트리 네스터룩) 학습, 해당 책을 공부하며 코드를 이 레포스트리에 정리하고 학습한 내용을 READ.MD에 작성하고자 한다.

## 몇가지 개념 

### 이상한 재귀 템플릿 패턴(Curiously Recurring Template Pattern, CRTP)

    struct Foo : SomeBase { ... }

 > 자기 자신을 베이스 템플릿 인자로 상속받는 것 

    template struct SomeBase{
	    void foo(){
		    for(auto& item : *static_cast<Derived*>(this)){ ... }}};

> 베이스 클래스의 구현부에서 타입이 지정된 this포인터를 사용할 수 있게 된다. 예를 들어 SomeBase를 상속받는 모든 서브 클래스들이 begin()/end() 메서드 쌍을 구현한다고 가정해 보자. SomeBase의 메서드 안에서 서브 클래스의 객체를 순회할 수 있을까? CRTP를 적용하면 this를 서브 클래스 타입으로 캐스팅할 수 있다. 

### 첨가(Mixin) 상속

    template struct Mixin : T { ... }

 > C++ 에서는 클래스를 정의할 때 자기 자신을 템플릿 인자로 할 수 있다. 이러한 방식을 첨가 상속(mixin inheritance)이라고 한다. 첨가 상속을 이용하면 계층적으로 여러 타입을 합성할 수 있다. 예를 들어 Foo<Bar\<Baz\>> x;와 같이 변수를 선언하면 새로운 타입 FooBarBaz를 따로 구현하지 않고도 세 가지 타입의 속성을 가지게 할 수 있다.

### 속성

    class Person{ 
	    int age_;
	public:
		int get_age() const { return age_; }
		void set_age(int value) { age_ = value; }
		__declspec(property(get=get_age,put=set_age)) int age; };

 > get/set 메서드를 가지는 클래스 내부 변수를 보통 그 클래스의 속성이라 부른다. 많은 프로그래밍 언어들이 속성을 언어 자체의 내장 기능으로 제공한다. 하지만 C++은 그렇지 않다. 하지만 컴파일러에서 비표준적인 방법으로 지원하고 있다.

## SOLID 디자인 원칙

### 단일 책임 원칙(Single Reponsibility Principle, SRP)
> **각 클래스는 단 한가지의 책임을 부여받아, 수정할 이유가 단 한가지여야 한다.**  SRP를 위배하는 안티 패턴의 극단적인 예로 전지전능 객체가 있다. 전지전능 객체는 가능한 많은 기능을 담아 하나의 괴물 같은 클래스를 이룬다. 

### 열림-닫힘 원칙(Open-Closed Principle, OCP)
> **타입이 확장에는 열려 있지만 수정에는 닫혀 있어야 한다.** 

### 리스코프 치환 원칙(Liskov Subsitution Principle, LSP)
> **어떤 자식 객체에 접근할 때 그 부모 객체의 인터페이스로 접근하더라도 아무런 문제가 없어야 한다.** 즉, 자식객체를 그 부모 객체와 동등하게 취급할 수 있어야 한다.

### 인터페이스 분리 원칙(Interface Sergregation Principle, ISP)
> **필요에 따라 구현할 대상을 선별할 수 있도록 인터페이스를 별개로 두어야 한다**

### 의존성 역전 원칙(Dependency Inversion Principle, DIP)
> **상위 모듈이 하위 모듈에 종속성을 가져서는 안 된다. 양쪽 모두 추상화에 의존해야 한다.**  **추상화가 세부 사항에 의존해서는 안된다. 세부 사항이 추상화에 의존해야 한다.** 즉, 종속성이 실 구현 타입이 아니라 인터페이스 또는 부모 클래스에 있어야 한다는 것을 말한다.

# 생성 패턴
 > 생성 패턴들은 객체의 생성과 관련된 공통적인 접근 방법들을 담는다.

## C++의 기본적인 부분

### 스택 할당
> 스택 메모리를 할당받아 객체를 생성하는 것을 말한다. 이러한 객체는 스코프를 벗어날 때 자동으로 소멸된다. 명시적으로 객체를 소멸시키지 않는다면 객체가 대입된 변수가 스코프에서 벗어나는 마지막 순간에 해당 객체의 소멸자가 호출된다.

### 힙 할당
> 원시 포인터를 이용하여 Foo\* foo = new Foo와 같이 객체를 생성하면 힙 메모리에 객체가 할당된다. 이렇게 인스턴스를 생성하면 메모리 해제를 비롯한 뒷정리를 누가 할 것인지가 불분명한 문제가 생긴다. 이 부분은 온전히 개발자에게 맡겨져 있다.

### unique 포인터(unique_ptr)
> "독점" 포인터라고도 한다. 이 스마트 포인터는 힙에 할당된 포인터를 받아, 더 이상 참조하는 곳이 없을 때 자동으로 소멸 작업이 이루어지도록 관리해준다. unique포인터는 "독점"이라는 이름이 의미하듯이 복제할 수 없고 다른 함수에 넘겨줄 수도 없다. 단, 원본과 그 제어권까지 통째로 이양하는 것은 가능하다.

### shared 포인터(shared_ptr)
> "공유" 포인터라고도 한다. 이 스마트 포인터는 힙에 할당된 포인터를 받아 코드 안의 다른 곳과 포인터를 공유하고 그 어느 부분에서도 더 이상 참조하는 곳이 없을 때 자동으로 소멸되도록 관리해준다.

### 약한 포인터(weak_ptr)
> "간접" 포인터라고도 한다. 이 포인터는 소유 개념이 없다. 단지 shared_ptr에 의해 관리되는 포인터를 간접적으로 참조한다. 실제 객체에 참조할 때는 shared_ptr로 변환해야만 한다. 이 포인터는 shared_ptr간에 순환 참조가 일어나는 것을 막기 위해 사용된다.

## 빌더
> 빌더 패턴은 생성이 까다로운 객체를 쉽게 처리하기 위한 패턴이다. 즉, 생성자 호출 코드 단 한줄로 생성할 수 없는 객체를 다룬다. 그러한 타입의 객체들은 다른 객체들의 조합으로 구성되거나, 상식적인 것을 벗어난 까다로운 로직을 필요로 한다. 이러한 객체를 생성하는 코드는 따로 분리되어 관리될 필요가 있다.

### 시나리오
> 웹페이지를 그리기 위한 컴포넌트들을 생성해야 한다고 하자. 먼저, 단순하게 단어를 나타내는 항목 두 개를 html의 비순차 리스트 태그로 출력해 보자.

    struct HtmlElement{
    	string name;
    	string text;
    	vector<HtmlElement> elements;
    	HtmlElement() {}
    	HtmlElement(const string& name, const string& text) : name(name), text(text) {}
    	string str(int indent = 0) const { ... } };
    
    string words[] = { "hello", "world" };
    HtmlElement list("ul", ""};
    for(auto w : words)	list.elements.emplace_back(HtmlElement("li",w));
    printf(list.str().c_str());

> 이 코드는 OOP에 기반하여 항목 리스트를 표현한다. 양식을 제어하기가 좀 더 쉬우면서도 목적하는 출력을 할 수 있다. 하지만 각각의 HtmlElement를 생성하는 작업이 그렇게 편리하다고 볼 수는 없다. 이 부분을 빌더 패턴을 활용하여 개선 할 수 있다.

### 단순한 빌더

    struct HtmlBuilder{
    	HtmlElement root;
    	HtmlBuilder(string root_name){ root.name = root_name; }
    	void add_child(string child_name, string child_text){
    		HtmlElement e{ child_name, child_text };
    		root.elements.emplace_back(e); }
    	string str(){ return root.str(); }
    
    HtmlBuilder builder{ "ul" };
    builder.add_child("li", "hello");
    builder.add_child("li", "world");
    cout << builder.str() << endl;

> 빌더 패턴은 단순하게 개별 객체의 생성을 별도의 다른 클래스에 위임한다. 위 코드는 빌더 패턴의 첫 번째 구현 시도이다. HtmlBuilder는 HTML 구성 요소의 생성만을 전담하는 클래스이다. add_child() 메서드는 현재 요소에 자식 요소를 추가하는 목적으로 사용된다. 그런데 add_child() 메서드는 리턴 값은 사용되는 곳 없이 void로 선언되어 있다. 리턴 값을 활용하면 좀 더 편리한 흐름식 인터페이스(fluent interface) 스타일의 빌더를 만들 수 있다.

### 흐름식 빌더

    HtmlBuilder& add_child(string child_name, string child_text){
    	HtmlElement e{ child_name, child_text };
    	root.elements.emplace_back(e);
    	return *this;
    }
    
    builder.add_child("li","hello").add_child("li","world");

> 빌더 자기 자신이 참조로서 리턴되기 때문에 메서드들이 꼬리를 무는 호출이 가능해진다. 이러한 형태로 호출하는 것을 흐름식 인터페이스(fluent interface)라고 부른다. 

### 의도 알려주기	

    protected:
    	HtmlElement() {}
    	HtmlElement(const string& name, const string& text) : name{name}, text{text} {}

> 그런데 사용자가 빌더 클래스를 사용해야 한다는 것을 어떻게 알 수 있을까? 한가지 방법은 빌더를 사용 안하면 객체 생성이 불가능하도록 강제하는 것이다. 위는 그렇게 하는 방법이다.


    ...
    private:
    	operator HtmlElement() const { return root; }
    	HtmlElement root;
    ...
    HtmlElement e = HtmlElement::build("ul")
    	.add_child("li", "hello")
    	.add_child("li", "world")
    cout << e.str() << endl;

> 최종 목적은 HtmlElement을 만드는 것이지 HtmlElement의 빌더를 만드는 것이 아니다. 따라서 가장 핵심적인 부분은 최종 값을 생성하는 "HtmlElement" 연산자의 구현이다.

### 그루비-스타일 빌더
> Groovy, Kotlin 등과 같은 프로그래밍 언어들은 도메인에 특화된 언어의 생성을 지원한다. 즉, 어떤 절차를 해당 응용의 성격에 맞추어 쉽게 기술할 수 있도록 새로운 언어를 정의할 수 있는 문법적 기능이 제공된다. 이 부분에서는 C++도 뒤떨어질 것이 없다. 초기화 리스트 기능을 이용하면 일반적인 클래스를 이용하여 HTMl에 호환되는 DSL을 효과적으로 만들 수 있다.  [소스 구현](https://github.com/DahamChoi/modern_cpp_designpattern/tree/main/creational/builder)


### 컴포지트 빌더
> 빌더에 관련된 마지막 예로 객체 하나를 생성하는데 복수의 빌더가 사용되는 경우를 살펴보자. 두 종류의 데이터 주소와 직업정보가 있는 Person 클래스가 있다고 생각해보자. 빌더를 각 정보마다 따로 두고 싶다면 API를 어떻게 만드는 것이 가장 편리할까? 이를 위해 콤포지트 빌더를 고안한다. 하나가 아니라 4개의 클래스로 빌더를 정의한다.

    class PersonBuilderBase{
    protected:
    	Person& person;
    	explicit PersonBuilderBase(Person& person) : person{ person } {}
    public:
    	operator Person(){ return std::move(person); }
    	PersonAddressBuilder lives() const;
    	PersonJobBuilder works() const; };

> 첫 번째 빌더 클래스는 PersonBuilderBase이다. 
> 1. 참조 변수 person은 현재 생성되고 있는 객체에 대한 참조를 담는다. Person의 실제 저장소가 이 클래스 안에 없다는 점이 대단히 중요하다. 베이스 클래스는 단지 참조만 가질 뿐, 생성된 객체는 가지지 않는다.
> 2. 참조 대입을 하는 생성자는 protected로 선언하여 그 자식 클래스들에서만 이용할 수 있게 한다.
> 3. operator Person을 통해 이동 생성자를 이용한다.
> 4. lives()와 works() 함수는 하위 빌더의 인터페이스를 리턴한다. 하위 빌더들은 각각 주소와 직업 정보를 초기화한다.


    class PersonBuilder : public PersonBuilderBase{
    	Person p;
    public:
    	PersonBuilder() : PersonBuilderBase{p} {} };


> 두 번째 빌더 클래스는 PersonBuilder이다. 여기서는 실제 객체가 생성된다. 이 클래스는 사실 빌더 베이스를 상속받을 필요가 없지만 빌더 구동 절차를 초기화하기 쉽도록 편의상 그렇게하고 있다.


    class PersonAddressBuilder : public PersonBuilderBase {
    	typedef PersonAddressBuilder self;
    public:
    	explicit PersonAddressBuilder(Person& person) 
	    	: PersonBuilderBase{ person } {}
    	self& at(std::string street_address){ 
    	person.street_address = street_address; return *this; }
    	self& with_postcode(std::string post_code) { ... }
    	self& in(std::string city){ ... } };

> 세 번째와 네 번째 빌더 클래스는 하위 빌더 클래스인 PeronAddressBuilder와 PersonJobBuilder이다. 
> 코드에서 보여지는 바와 같이 PersonAddressBuilder는 Person의 주소를 생성하는데 풀루언트 인터페이스 스타일을 지원한다. PersonAddressBuilder는 PersonBuilderBase를 상속받고 있고 Person의 참조를 인자로 하여 베이스의 생성자를 호출한다.


    Person p = Person::create()
    	.lives().at("123 London Road")
    			.with_postcode("SW1 1GB")
    			.in("London")
    	.works().at("PragmaSoft")
    			.as_a("Consultant")
    			.earning(10e6)

> 이제 사용자가 어떻게 이 빌더를 사용할 수 있는지 확인해보자.
### 요약

> 빌더 패턴의 목적은 여러 복잡한 요소들의 조합이 필요한 객체를 생성해야 하거나 또는 여러개의 다양한 객체 집합을 생성해야 할 때 객체 생성만을 전담하는 컴포넌트를 정의하여 객체 생성을 간편하게 하는 것이다. 빌더 패턴에서 빌더의 특징을 정리하면 다음과 같다.
> 1. 흐름식 인터페이스를 이용하면 복잡한 생성 작업을 한 번의 호출 체인으로 처리할 수 있다.
> 2. 사용자에게 빌더 API의 사용을 강제하기 위해, 타겟 객체의 생성자를 외부에 접근하지 못하게 설정하고 빌더에 static create() 함수 추가하여 생성된 객체를 리턴하게 한다.
> 3. 적절한 연산자를 정의하여 객체 자체적으로 빌더의 사용을 강제할 수도 있다.
> 4. 유니폼 초기화 문법을 이용하면 C++에서도 그루비 스타일 빌더를 만들 수 있다. 이러한 접근 방법으로 매우 일반적이며 다양한 DSL을 만들 수 있다.
> 5. 빌더 하나의 인터페이스가 여러 하위 빌더를 노출할 수도 있다. 상속과 흐름식 인터페이스를 요령있게 활용하면, 여러 빌더를 거치는 객체 생성을 쉽게 할 수 있다.
