# LearnAndroidDev
Week1 Compose Internal 
### Prelude
- Jetpack Compose Internals는 컴파일러 및 런타임의 디테일한 내용에 매우 중점을 두고 있어, 목표로 하시는 플랫폼에 있어서 매우 독립적인 경험을 선사
### Composable functions
> Composable 함수는 Jetpack Compose의 가장 기본이 되는 요소이며, Composable 트리 구조를 작성하는 데 사용됩니다. 여기서 저자는 의도적으로 “트리(trees)“라고 이야기하는데, 그 이유는 Compose Runtime(컴포즈 런타임)이 Composable 함수를 메모리에서 큰 트리의 일원인 하나의 노드(node)로서 이해하고 나타낼 수 있기 때문입니다.

> @Composable 어노테이션을 사용함으로써, 우리는 컴파일러에게 이 함수가 본질적으로 데이터1를 하나 의 노드(node)로 변환하여 Composable 트리(tree)에 기재하겠다는 의도를 전달

> 트리에 요소를 삽입하기 위해 기재된 일련의 동작(action)이라고 볼 수 있습니다. 우리는 이 동작을 함수 실행의 부수 효과(side effect)

* Composable 함수를 실행하는 유일한 목적은 트리의 인메모리 표현(in‐memory representation) 을 만들거나 업데이트하는 것입니다. Composable 함수는 읽은 데이터가 변경될 때마다 다시 실행 되므로 나타내는 메모리 구조를 항상 최신 상태로 유지

```
용어정리
 Composition: Composable 함수를 실행하고 UI를 나타내는 트리 구조를 출력하는 행위.
 Recomposition: Composition을 다시 실행하여 레이아웃을 업데이트하는 일련의 과정.
```

### Calling context
> Kotlin 컴파일러 플러그인인 Compose Compiler는 일반적인 컴파일러 실행단계 중에 동작,  Kotlin 컴파일러가 액세스할 수 있는 모든 정보에 액세스할 수 있습니다.

> Composable 함수의 중간 표현인 IR4을 가로채고 변환하여 원본 소스의 모든 Composable 함수에 추가적인 정보를 부여

> Composer 매개변수의 인스턴스는 런타임에 주입되며, 모든 하위 Composable 호출로 전달되므로 트리의 모든 수준에서 접근 가능

> Composable 함수는 오로 지 다른 Composable 함수에서만 호출될 수 있습니다. 실제로 이 호출 컨텍스트(calling context)는 필수적이며, 이는 트리가 오직 Composable 함수로 구성되도록 보장하고, Composer가 트리를 따라 하향 전달될 수 있도록 합니다.

> Composer는 개발자가 작성하는 Composable 코드와 Compose Runtime 간의 중재자 역할을 합니 다. Composable 함수는 트리에 대한 변경 사항을 전달하고, 런타임 시에 트리의 형태를 빌드하거나 업데이트하기 위해 Composer를 사용

```
용어정리
 Intermediate Representation: Kotlin Compile 과정에서 최적화 및 구현
```

### Idempotent
> 동일한 입력 매개변수 를 사용하여 Composable 함수를 여러 번 다시 실행하더라도 동일한 트리가 생성되어야 합니다.

> recomposition이 의미하는 바는, 입력값이 변경될 때 마다 Composable 함수를 다시 실행하여 업데이트된 정보를 방출시키고 트리를 업데이트하는 작업

> 동일한 입력값에 대한 결과는 이미 메모리에 적재되어 있으므로 Compose는 다시 실행할 필요가 없고, 결과적으로 생략할 수 있습니다

### 통제되지 않은 사이드 이펙트 방지 (Free of uncontrolled side effects)
> 사이드 이펙트(side effect)6는 호출된 함수의 제어를 벗어나서 발생할 수 있는 예상치 못한 모든 동작 을 의미(외부 요인에도 의존)

> 사이드 이펙트는 Compose 내에서 아무 통제를 받지 않고 여러 번 실행될 수 있습니다. Composable 함수가 사이드 이펙트를 실행한다면 매 함수 호출 시마다 새로운 프로그램 상태를 생성할 수 있으므로, Compose Runtime에게 필수적인 멱등성을 따르지 않게 됩니다.

> 사이드 이펙트는 Composable 함수에서 이상적이지 않습니다. 우리는 모든 Composable 함수를 stateless(무상태, 상태를 보존하지 않음)하게 만들려고 노력해야 합니다

> 이펙트 핸들러는 사이드 이펙트가 Composable의 라이프사이클을 인식하도록 하여, 해당 라이프사 이클에 의해 제한되거나 실행될 수 있게 합니다

> 이펙트 핸들러는 Composable 함수 내에서 아무런 제어를 받지 못하고 직접적으로 이펙트가 호출되는 것을 방지하는 역할을 수행

### 재시작 가능 (Restartable)
> Compose는 메모리 내 표현을 항상 최신 상태로 유지하기 위해 트리의 어떤 노드를 재시작할지 선 택적으로 판단합니다. Composable 함수는 관찰하는 상태(state)의 변화에 기반하여 반작용적으로 재실행되도록 설계

> Compose Compiler는 일부 상태(state)를 읽는 모든 Composable 함수를 찾아 Compose Runtime에게 재시작하는 방법을 가르치는 데 필요한 코드를 생성

### Fast execution
>. Composable은 단순히 인메모리 구 조를 구축 및 업데이트하기 위한 데이터를 방출할 뿐입니다. 이 매커니즘이 Composable을 더욱 빠르게 만들며, 런타임이 아무 문제없이 해당 함수를 여러번 실행할 수 있도록 합니다.
-> 인메모리 구조 (RAM(Random Access Memory)에 존재하는 데이터 구조를 의미) -> 데이터 emitting -> 트리 저장

### 위치 기억법 (Positional memoization)
>위치 기억법은 함수 메모이제이션(memoization)9의 한 형태입니다. 함수 메모이제이션은 함수가 입력값에 기반하여 결과를 캐싱하는 기법

>Composable 함 수는 소스 코드 내 호출 위치에 대한 불변의 정보를 가지고 있습니다. Compose Runtime은 동일한 함수가 동일한 매개변수 값으로 다른 위치에서 호출될 때, 동일한 Composable 부모 트리 내에서 고유한 다른 ID를 생성합니다

>Composable의 정체성은 recomposition을 거쳐도 유지되므로, Compose Runtime이 이 구조를 활용하여 Composable이 이전에 호출되었는지 여부를 파악하고 가능한 경우 생략할 수 있습니다

>Compose는 key라는 Composable 함수를 제공하는데, 우리는 이 함수를 이용하여 Composable 함수에게 명시적인 키 값을 지정

>Compose Compiler에 의해 재시작이 가능하다고(Restartable) 추론된 모든 Composable 함수는 recomposition을 생략할 수 있어야 하며, 그 말인즉 인메모리에 자동적으로 기억됨을 의미합 니다. Compose는 이러한 메커니즘으로 구현되어 있습니다.

```
용어정리
메모이제이션(memoization): 이전에 계산한 값을 메모리에 저장함으로써 동일한 계산의 반복 수행을 줄임, 캐싱의 한 유형.
순수 한(결정론적10) 함수:주어진 조건들을 만족하는 유일 해가 존재한다는 가정하에서의 문제 접근 방법.
```

### Suspend 함수와의 유사성 (Similarities with suspend functions)
>Continuation 인터페이스는 실행을 중단하고 재개하는 것에 매우 구체적이므로, 콜백 인터페이스로 모델링되어 있으며, Kotlin은 실행 지점 간 점프를 수행하고, 다양한 중단점을 조율하고, 중단점 사이에서 데이터를 공유하는 등 필요한 모든 시스템과 함께 기본적인 구현을 생성합니다. 그러나, Compose의 사용 사례는 이와 매우 다릅니다. Compose의 목표는 런타임에서 다양한 방법으로 최적화될 수 있는 대규모의 함수 호출 그래프에서 인메모리 표현을 생성하는 것이기 때문입니다.

### Composable 함수의 색깔 (The color of Composable functions)
> Composable 함수는 프로그램 로직을 작성하기 위해 설계된 것이 아니라 노드 트리의 변경사항을 기술하기 위한 것
> 
> Kotlin에서 suspend 수정자를 함수에 추가함으로써 매우 관용적이고 풍부한 방식으로 비동기 non‐ blocking 프로그램을 모델링할 수 있습니다. 반면, @Composable 어노테이션은 표준함수가 재시작할 수 있는(restartable), 생략할 수 있는(skippable), 그리고 반응형(reactive)과 같이 Kotlin의 표준함 수에는 없는 기능을 가지도록 만듭니다.

### Composable 함수 타입 (Composable function types)
>@Composable 어노테이션은 컴파일 시점에 함수의 타입을 효과적으로 변경

>수의 구문 (syntax)적 관점에서, Composable 함수의 타입은 @Composable (T) ‐> A 입니다. 여기서 A는 Unit일 수도 있고, 함수가 값을 반환하는 경우(예를 들어 remember) 다른 타입일 수도 있습니다. 개발자들은 Kotlin에서 일반적인 람다를 선언하는 것처럼 Composable 람다를 선언할 수 있습니다.
>
>언어의 관점에서 볼 때, 타입은 컴파일러에 정보를 제공하여 빠른 정적 검증을 수행하고, 때로는 편리한 코드를 생성하며, 런타임에 의해 활용되는 데이터 사용 방식을 제한 및 정제하기 위하여 존재합니다. @Composable 어노테이션은 런 타임 시 Composable 함수의 유효성을 검사하고 사용하는 방법을 변경하는데, 이것이 바로 Composable 함수가 Kotlin의 표준 함수와 다른 타입으로 간주되는 이유입니다.

###  Compose 컴파일러 (TheCompose compiler)
> Compose Compiler는 실제의 Kotlin 컴파일러 플러그인입니다. 이는 라이브러리가 Kotlin 컴파일 단계 내에서 진행되는 컴파일 타임 작업을 수반할 수 있게 하여, 코드의 형태에 있어 보다 관련성 있는 정보에 접근하고 전체 프로세스를 가속화할 수 있습니다. kapt는 컴파일 이전에 실행해야 하지만, 컴파일러 플러그인은 컴파일 과정에 직접 내장되어 있습니다.

> Compose Compiler는 Compose Runtime이 요구하는 대로 Composable 함수를 마음대로 변형시킬 수 있습니다.

### Compose 어노테이션들 (Compose annotations)
>
Compose Compiler는 Kotlin 컴파일러의 프론트엔드에서 후크9 및 확장점을 활용하여 강제하고자 하는 제약 사항이 충족되고 있는지, 타입 시스템이 Composable 함수나 선언 및 표현식을 적절히 일반적인(Composable 함수가 아닌) 함수들과는 다르게 처리하고 있는지를 확인합니다.

>그 외에도, Compose는 특정 상황에서 추가적인 검사와 다양한 런타임 최적화 또는 “숏컷10 “을 활성화하기 위해 추가적인 어노테이션들을 제공합니다. 모든 Compose 어노테이션들은 Compose Runtime 라이브러리에 의해 제공됩니다.

### @Composable
> Compose Compiler와 어노테이션 프로세서의 가장 큰 차이점은, Compose의 경우 실제로 어노 테이션이 붙어있는 선언이나 표현식을 변형한다
> @Composable을 통해 선언이나 표현식의 타입을 변경하는 것은 대상에게 “메모리“를 부여하는 것을 의미합니다. 즉, remember를 호출하고 Composer 및 슬롯 테이블을 활용할 수 있는 능력을 의미합니 다. 또한, Composable의 본문 내에서 구동된 이펙트들(effects)이 준수할 수 있는 라이프사이클을 제공합니다.

### @DisallowComposableCalls
>함수 내에서 Composable 함수의 호출이 발생하는 것을 방지하기 위해 사용

>remember 함수는 calculation 블록에 의해 제공된 값을 기억합니다. 이 calculation 블록은 최초의 composition 단계에서만 수행되며, 이후의 모든 recomposition 단계에서는 항상 이미 계산된 값을 반환

### @NonRestartableComposable
> 인라인 된 Composable 함수나 반환 타입이 Unit이 아닌 Composable 함수는 재시작할 수 없으므로, 모든 Composable이 기본적으로 재시작 가능한 것은 아닙니다
