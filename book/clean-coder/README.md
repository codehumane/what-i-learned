
# 프로의 마음가짐

## 책임감을 가져라

- '프로'라는 단어에 '책임'을 포함시켜 이야기.
- 한 가지 예로, 자신이 실수로 오류를 만들고, 이에 회사가 손해를 입음. 프로라면 손해배상을 함.
- 개인적으로는 비판의 여지가 많은 내용이라 생각.
- 한편, 아래의 사례는 인상적.

> 돌이켜보면 야간 작업을 테스트하지 않고 선적한 일은 무책임한 것이었다. 테스트를 소홀히 한 이유는 제때 선적했다고 떠벌리고 싶었기 때문이다. 그게 내 체면을 세우는 일이었다. 고객이나 회사는 뒷전이었다. 관심을 가졌던 건 내 평판뿐이었다. 책임감을 가지고 미리 톰에게 테스트가 끝나지 않아서 제때 선적할 수 없다고 말했어야 옳았다. 힘든 결정이었을 테고, 톰은 틀림없이 화를 냈을 것이다. 하지만 고객은 데이터를 잃어버리지 않았을 테고 서비스 매니저들도 전화를 하지 않았을 것이다.

## 해를 끼치지 말아라

- 이런 책임감을 위해 소프트웨어 개발자가 할 수 있는 것은 뭘까?
- 기능과 구조. 이 2가지에 해를 끼치지 않는 것을 이야기 함.

### 기능에 해를 끼치지 말아라

- 기능에 해를 끼치지 말라는 이야기는 오류를 만들지 말라는 것.
- 복잡성 때문에 오류가 생길 수도 있지만, 그렇다고 해서 오류가 괜찮다는 것은 아님.
- 오류 비율을 0에 가깝게 만드는 것이 개발자의 책임.
- 이를 위한 방법으로 테스트 하고 또 테스트 해야 함을 강조.
- 시간이 부족하다면, 테스트 자동화 등의 노력을 수반.
- 개인적으로는 '채'를 많이 만들어야 한다고 생각.
- 그리고 이런 채들은 AND 조건으로 만들어야 함.
- 9/10 확률로 오류를 거르는 채가 5개만 있어도,
- (1/10)^5의 확률로 문제가 줄어듦.

### 구조에 해를 끼치지 말아라

- 소프트웨어는 변함.
- 변할 수 있어야 수익 모델이 유지.
- 변할 수 있으려면 구조가 유연해야 함.
- 터무니 없는 비용 지불 없이도 변경할 수 있어야 하는 것.
- 그렇지 않다면 점점 더 느려지고, 변경마다 더 많은 문제가 발견될 것.
- 구조에 문제가 없는지는 실제로 변경해 보면서 알게 됨.
- 그러면서 발견한 문제점들을 개선. 지속적으로.

## 직업 윤리

- 자신의 경력은 자신이 책임져야 함을 이야기.
- 책을 사주지 않고, 컨퍼런스를 보내주지 않는 지금의 회사를 탓할 이유가 없는 것.
- 한편, 한 주에 60시간을 경력을 위해 사용하라고 이야기도.
- 40은 회사 업무를 위해, 20시간은 연습하고 공부하며 경력에 도움되는 곳에.
- <자본론>의 노동과 자본의 불평등 교환이 생각나는 대목.
- 정확하고 명시적인 계산을 하지는 않지만, 면접이나 평가 등의 과정에서 종종 이 20시간이 고려되기도.

## 끊임없이 배우기

- 전산 분야 지식을 익히라면서, 철학자 산타야나의 저주를 소개.

> 과거를 기억하지 못하는 자, 그 과거를 반복하는 저주를 받을지니...

- 품새(kata)도 이야기.
- 저자는 볼링 점수 계산이나 인수분해 등의 간단한 프로그래밍 훈련을 반복.
- 매일 품새를 한두 개씩 풀고, 다양한 언어로 같은 문제를 풀기도 함.
- 단축키에 익숙해지는 등의 노력도.
- 아침운동 10분이나 저녁 마무리 운동 10분으로 생각한다고 함.
- 멘토링도 강조. 이것 또한 다시 한 번 혼자 고민해 볼 일.

# 예라고 말하기

아래 대화의 대답들은 모호함.

```
마지: 피터, 평가 엔진 수정을 금요일까지 끝낼 수 있나요?
피터: 가능하다고 생각해요.
마지: 문서 작업까지 포함해서요?
피터: 문서 작업까지 끝내도록 노력해볼게요.
```

저자는 이렇게 응답하기를 권장.

```
마지: 피터, 평가 엔진 수정을 금요일까지 끝낼 수 있나요?
피터: 가능할 겁니다. 하지만 월요일에 끝날지도 몰라요.
마지: 문서 작업까지 포함해서요?
피터: 문서 작업은 몇 시간 걸리니까 월요일에 끝낼 수도 있어요. 하지만 화요일까지 늦어질지도 몰라요.
```

좀 더 정직했고, 불확실성이 드러났으며, 마지가 이 불확실성을 감수할지를 결정하게 할 수 있다.

그런데, 뒤이어 소개되는 가정들과 그에 따른 대화들이 더 흥미로움. 일정 약속을 위해 좀 더 확실한 대답이 필요한 경우, 수정과 문서 작업이 금요일까지 모두 끝나야 하는 경우 등등이 나옴. 이 때, 개발자로서 리팩토링이나 테스트 코드 작성을 포기해야 하는지에 대한 얘기도.
