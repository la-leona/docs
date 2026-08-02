# Rust Basics

Kimi: Rust는 **안전성과 속도**를 동시에 추구하는 시스템 프로그래밍 언어입니다. PowerShell과 비교하면서 핵심 문법을 정리해 드리겠습니다.

---

## 1. 변수와 타입

### `let`으로 선언, 기본적으로 **불변(immutable)**

```rust
let name = "Rust";      // 문자열 (&str)
let count = 10;         // i32 (정수, 기본)
let pi = 3.14;          // f64 (실수, 기본)
let is_ok = true;       // bool
```

### 가변 변수는 `mut` 필수

```rust
let mut score = 0;      // 가변
score = 100;            // OK

let score = 0;
score = 100;            // ❌ 컴파일 에러
```

### 타입 명시

```rust
let age: i32 = 25;
let price: f64 = 19.99;
let flag: bool = true;
```

| Rust 타입 | 설명 | PowerShell 대비 |
|---|---|---|
| `i32` | 32비트 정수 | `[int]` |
| `i64` | 64비트 정수 | `[long]` |
| `f64` | 64비트 실수 | `[double]` |
| `bool` | true/false | `[bool]` |
| `String` | 소유권 있는 문자열 | `[string]` |
| `&str` | 문자열 슬라이스 (참조) | 문자열 리터럴 |
| `Vec<T>` | 동적 배열 | `[System.Collections.Generic.List]` |
| `Option<T>` | 값이 있거나 없거나 | `$null` 체크 |

---

## 2. 함수

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b               // 마지막 표현식이 반환값 (세미콜론 없음!)
}

fn greet(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let sum = add(5, 3);
    greet("World");
}
```

- `fn` = 함수 선언
- `-> 타입` = 반환 타입
- **마지막 줄에 `;` 없으면 그게 반환값**
- `return` 키워드도 쓸 수 있지만, Rust에서는 생략이 관례

---

## 3. 제어문

### if / else

```rust
let num = 10;

if num > 0 {
    println!("양수");
} else if num < 0 {
    println!("음수");
} else {
    println!("0");
}
```

> Rust에서는 **조건식에 반드시 `bool`**이어야 함. `if 1 { }` 같은 건 안 됨.

### if를 표현식으로 (삼항 연산자 대체)

```rust
let result = if score >= 60 { "합격" } else { "불합격" };
```

### 반복문

```rust
// for (범위)
for i in 0..5 {          // 0,1,2,3,4
    println!("{}", i);
}

// while
let mut n = 0;
while n < 5 {
    n += 1;
}

// loop (무한 루프)
loop {
    println!("계속");
    break;
}
```

---

## 4. 소유권 (Ownership) — Rust의 핵심

Rust는 **가비지 컬렉터 없이** 메모리를 안전하게 관리합니다. 그 비결은 **소유권 규칙**입니다.

### 규칙 3가지

1. 값은 **한 번에 하나의 변수**만 소유할 수 있다
2. 소유자가 **스코프를 벗어나면** 값은 자동 해제된다
3. 소유권은 **이동(move)**하거나 **빌림(borrow)**할 수 있다

### 예시

```rust
let s1 = String::from("hello");
let s2 = s1;              // 소유권이 s1 → s2로 이동 (move)

// println!("{}", s1);   // ❌ 컴파일 에러! s1은 더 이상 유효하지 않음
println!("{}", s2);       // OK
```

### 빌림 (참조)

```rust
let s = String::from("hello");

let len = calculate_length(&s);   // & = 불변 참조 (빌림)
println!("{}의 길이: {}", s, len); // s는 여전히 사용 가능!

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

### 가변 참조

```rust
let mut s = String::from("hello");
change(&mut s);           // &mut = 가변 참조

fn change(s: &mut String) {
    s.push_str(", world");
}
```

> **가변 참조는 동시에 하나만** 존재할 수 있습니다. 데이터 경쟁(race condition)을 원천 차단.

---

## 5. 구조체 (Struct)

PowerShell의 `[pscustomobject]`처럼 여러 필드를 묶는 타입입니다.

```rust
struct User {
    name: String,
    age: u32,
    active: bool,
}

fn main() {
    let user = User {
        name: String::from("Kim"),
        age: 30,
        active: true,
    };
    
    println!("{}님은 {}살", user.name, user.age);
}
```

### 메서드

```rust
impl User {
    fn greet(&self) {
        println!("안녕하세요, {}입니다", self.name);
    }
}

let u = User { name: "Kim".to_string(), age: 30, active: true };
u.greet();
```

---

## 6. 열거형 (Enum) + 패턴 매칭

```rust
enum Status {
    Success,
    Error(String),
    Loading(u32),   // 로딩 퍼센트
}

fn handle(status: Status) {
    match status {
        Status::Success => println!("성공!"),
        Status::Error(msg) => println!("에러: {}", msg),
        Status::Loading(pct) => println!("{}% 로딩중...", pct),
    }
}
```

> `match`는 **모든 경우를 다루지 않으면 컴파일 에러** — 누락 방지

---

## 7. Option과 Result — null/예외 처리

Rust에는 **`null`이 없습니다**. 대신 `Option`과 `Result`를 씁니다.

### Option<T> — 값이 있을 수도, 없을 수도

```rust
let maybe: Option<i32> = Some(10);
// let maybe: Option<i32> = None;

match maybe {
    Some(v) => println!("값: {}", v),
    None => println!("값 없음"),
}

// 축약
if let Some(v) = maybe {
    println!("값: {}", v);
}
```

### Result<T, E> — 성공 또는 실패

```rust
use std::fs::File;

let result = File::open("test.txt");

match result {
    Ok(file) => println!("파일 열기 성공"),
    Err(e) => println!("에러: {}", e),
}

// ? 연산자: 성공이면 값을, 실패면 즉시 반환
let file = File::open("test.txt")?;
```

---

## 8. 벡터 (동적 배열)

```rust
let mut nums: Vec<i32> = Vec::new();
nums.push(10);
nums.push(20);

// 축약
let nums = vec![10, 20, 30];

for n in &nums {
    println!("{}", n);
}
```

---

## 9. 모듈과 use

```rust
// math.rs 파일
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

// main.rs
mod math;

fn main() {
    let sum = math::add(2, 3);
}
```

또는 `use`로 가져오기:

```rust
use math::add;
let sum = add(2, 3);
```

---

## 10. Tauri에서 자주 보는 Rust 문법

| 코드 | 의미 |
|---|---|
| `#[command]` | Tauri가 JS에서 호출할 수 있는 함수 표시 |
| `-> Result<T, String>` | 성공 시 T, 실패 시 에러 메시지 반환 |
| `.map_err(\|e\| e.to_string())?` | 에러를 문자열로 변환 후 전파 |
| `tauri::generate_handler![foo]` | Tauri에 함수 등록 |
| `tauri::generate_context!()` | tauri.conf.json 로드 |

---

## 요약

| 개념 | Rust | PowerShell |
|---|---|---|
| 변수 | `let mut x = 1` | `$x = 1` |
| 타입 | `i32`, `f64`, `String` | `[int]`, `[double]`, `[string]` |
| 함수 | `fn foo() -> i32 {}` | `function foo() {}` |
| 조건문 | `if x > 0 {}` | `if ($x -gt 0) {}` |
| 반복문 | `for i in 0..5 {}` | `foreach ($i in 0..4) {}` |
| null | `Option<T>` (Some/None) | `$null` |
| 예외 | `Result<T, E>` (Ok/Err) | `try/catch` |
| 객체 | `struct` + `impl` | `[pscustomobject]@{}` |

Rust는 **컴파일러가 실수를 미리 잡아주는** 언어입니다. 처음에는 소유권 때문에 에러가 많이 나지만, 익숙해지면 **메모리 누수 없이** 매우 빠른 프로그램을 만들 수 있습니다.

특정 문법(예: `Result` 처리, `async/await`, `trait`)을 더 깊게 알고 싶으시면 말씀해 주세요!
