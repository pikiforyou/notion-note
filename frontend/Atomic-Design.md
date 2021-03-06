# Atomic Design ?

![https://media.vlpt.us/images/seob/post/67838f3b-18c7-4372-93a0-da3bf08888e2/atomic-design-process.png](https://media.vlpt.us/images/seob/post/67838f3b-18c7-4372-93a0-da3bf08888e2/atomic-design-process.png)


뷰를 Atoms(원자) -> Molecules(분자) -> Organisms(유기체) -> Templates -> Pages 순으로 작은 것들을 만들고, 결합해 좀 더 큰 단위의 뷰를 만들어 나가는 디자인 시스템입니다.웹앱은 여러 페이지 단위로 이루어지고 페이지는 input, button, form 등의 태그들로 이루어져있습니다. 이를 원자, 분자, 유기체같은 생물학적인 개념으로 접근한 것입니다.

## 장점

1. 재사용 가능한 설계 시스템을 제공합니다.컴포넌트들을 혼합해 일관성 있고 재사용의 효율을 높이는 디자인을 할 수 있습니다.
2. 디자인을 쉽게 수정할 수 있습니다.컴포넌트가 단위별로 이루어져 큰 컴포넌트에서 작은 컴포넌트를 삭제, 추가, 수정하는 것으로 쉽게 수정할 수 있습니다.
3. 레이아웃을 이해하기 쉬어집니다.페이지를 처음부터 설계하는 시도가 있어, 페이지의 레이아웃의 이해가 오래가고, 팀 프로젝트 시 제 멋대로가 되는 스타일 가이드를 최소화시킵니다.

## 단점

1. 오랜 기간의 디자인 설계의 개념은 이상적이나 설계에 힘을 써야하는 장점이 오히려 단점이 됩니다.
2. 일관성이 떨어지는 결과 발생 위험성잘못된 디자인으로 컴포넌트들을 합치고 나눌 시, `기술 부채`와 `개발 기간이 증대`해 결국엔 일관성이 떨어지는 디자인의 결과가 발생할 것입니다.
3. 원자, 분자보다 익숙한 유기체보통 사람들은 큰 단위를 정하고 그 내용물로 작은 단위를 만드는 top-down 방식에 익숙합니다. 그래서 원자, 분자부터 만드는 bottom-up이 익숙하지 않아 학습과 훈련이 필요하다는 것입니다.

---

## 구성 요소

![https://media.vlpt.us/images/thsoon/post/b9aa9272-b181-4831-9844-79e3f8124b3c/image.png](https://media.vlpt.us/images/thsoon/post/b9aa9272-b181-4831-9844-79e3f8124b3c/image.png)

### Atoms

- 해당 설계의 최소 단위
- form, input ,button 같은 HTML의 태그나 최소의 기능을 가진 기능의 커스텀 태그 컴포넌트
- 설계에 따라 속성에 따른 스타일 주입이 들어갈 수 있습니다.
- Card System에서 제목, 내용, footer 들이 각각 이에 해당됩니다.

### Molecules

- Atom들을 최소의 역할을 수행할 수 있게 합한 그룹
- 입력을 받기 위한 form + label + input이 해당 됩니다.
- Card System에서 제목 + 내용 + footer들이 합쳐진 하나의 Card가 이에 해당됩니다.

### Organisms

- 배치를 위한 layout 단위로 하나의 인터페스를 형성하는 그룹
- header, navigation 등이 이에 해당됩니다.
- Card System에서 Card들이 Grid layout으로 형성된 집합이 이에 해당됩니다.

### Templates

- 실제 Organisms들을 레이아웃이나 데이터 흐름을 연결합니다.
- 클래스 시스템의 클래스로, 객체의 설계도, 페이지의 설계도입니다.

### Pages

- 정의된 Template에 데이터를 넣어 뷰를 완성시키는 단계입니다.
- 클래스 시스템의 인스턴스, 객체의 구현체, 페이지 설계도로 그린 페이지 그 자체입니다.
