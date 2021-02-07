# PerlinnoiseSampleCode_for_unity

## 펄린 노이즈란?
펄린 노이즈는 일련의 난수의 생성을 수행하고 제어하며 여러 노이즈가 점진적으로 변화되는 특성 때문에 여러 분야에서 무궁무진하게 사용할 수 있는 알고리즘이다. 주로 컴퓨터 그래픽스 분야에서 활용되고 있으며 얻을 수 있는 데이터가 제한적인 상황에서 AI에게 학습시켜 의료계열, 예술계열 등 여러 방면에서 많은 데이터를 증대할 수 있다.  
난수는 임의의 수, 즉 예측 불가능 한 수이기에 보통 제어할 수 없다고 생각할 것이다. 하지만 마치 랜덤처럼 보이나 사실은 랜덤이 아닌 값인 의사 난수를 활용한다면 이를 개발자의 의도대로 구현할 수 있다. 게임은 하나의 가상 시뮬레이션이기 때문에 의사 난수를 잘 이용한다면 컴퓨터 그래픽 뿐 아니라 게임에서 절차적 지형 생성(게임 지형 생성)이나 AI의 행동의 생성에 관여하여 더욱 현실과 비슷하게 구현할 수 있을 것이다.  
## 알고리즘
펄린 노이즈를 생성하기 위해서는 방법을 알아야 할 것이다. 기존의 고안자인 켄 펄린이 2002년에 개선된 펄린 노이즈를 발표한 이후로 펄린 노이즈를 생성하는 여러 방법이 제시되었으며 이 글에서는 간단한 1차원 펄린 노이즈만 생성하며 구조는 다음과 같다.
  
> 의사 난수 생성 -> 난수를 바탕으로 여러 옥타브 생성 -> 옥타브 보간 -> 펄린 노이즈 생성
  
아래에서 다시 다루겠지만 의사 난수는 시드 값을 바탕으로 생성되는데 시드 값의 제어는 이 글에서 다루지 않는다. 그렇기 때문에 생성된 모든 값은 주기성을 띄며 이것의 조절에 따라 다채로운 랜덤 값을 생성할 수 있다. - 사실 랜덤처럼 보이지만 인위적으로 생성된 값이며, 완전한 랜덤이 아니지만 완전한 랜덤이라 보아도 무방하다.  

### Creating Noise
XOR 시프트 연산(의사 난수 생성)을 통해 랜덤한 값을 생성한다. 예제에서는 같은 시드를 계속 사용하며, 시드에 따라 다른 값이 생성된다.

![Noise](https://user-images.githubusercontent.com/62614157/107154553-d5fca200-69b6-11eb-8af2-6a277e8822e7.gif)

### Noise Smoothing
이전에 Noise에서 생성된 Noise값들의 min과 max를 보간하여 smooth 작업을 통해 평균의 값을 얻을 수 있다. 이는 1차원 펄린 노이즈에서는 그 영향이 미미하나,  2, 3차원 펄린 노이즈에서는 fading 역할을 하여 애니메이션이나 이미지가 더욱 부드럽게 보이게 할 수 있다.

![SmoothNoise](https://user-images.githubusercontent.com/62614157/107154556-d85efc00-69b6-11eb-854c-680eedc551cf.gif)

### Interpolating
마지막으로 SmoothNoise로 생성된 값(옥타브)들을 보간한다면 여러 방면으로 활용할 수 있는 펄린 노이즈의 생성이 가능하다. 각 옥타브들을 잘 제어하여 보간한다면 더 세밀한 노이즈를 생성할 수 있고 이 부분들을 잘 적용한다면 각 동작들의 특성을 구현할 수 있을 것이다.

![PerlinNoise](https://user-images.githubusercontent.com/62614157/107154558-d9902900-69b6-11eb-8d58-ffc7a0a5af25.gif)

### Control of Seed
시드를 제어하기 위해서는 랜덤한 시드를 생성하는 방법이 필요한데, 위에서 언급한 것처럼 랜덤한 값을 만들기 위해 랜덤한 값이 필요한 딜레마에 빠지게 된다. 보통 해결하는 방법은 시스템에서 제공하는 노이즈 값이나 시간을 사용하는 방법 등을 이용하여 가짜 랜덤이지만 마치 진짜 랜덤인 것처럼 보이게 할 수 있다.  

## 샘플 코드
```csharp
 public static float Noise(int x)
    {
        x = (x << 13) ^ x;
        return ( 1.0f - ((x * (x * x * 15731 + 789221) + 1376312589) & 0x3f3f3f3f) / 1073741824.0f);
    }

public static float SmoothNoise(int x)
    {
        return Noise(x) / 2 + Noise(x-1) / 4 + Noise (x + 1) / 4;
    }

public static float Interpolate(float a, float b, float c, float d, float x)
    {
        float p = (d - c) - (a - b);
        float q = (a - b) - p;
        float r = c - a;
        float s = b;

        return p * (x * x * x) + q * (x * x) + r * x + s;
    }
	
public static float PerlinNoise(float x)
    {
        int floor = Mathf.FloorToInt(x);
        float t = x - floor;

        return Interpolate(SmoothNoise(floor - 1), SmoothNoise(floor), SmoothNoise(floor + 1), SmoothNoise(floor + 2), t);
    }
```

## 데모 적용 결과
펄린 노이즈의 생성까지 완료했다면 유니티에서 이를 실행시켜 볼 차례이다. 아래 사진은 유니티 스크립팅 API에서 Mathf.PerlinNoise의 예제 스크립트를 큐브 오브젝트에 적용한 것이다.  

![Example1](https://user-images.githubusercontent.com/62614157/107154559-dc8b1980-69b6-11eb-8874-3eb570404300.gif)

유니티 스크립팅 API에서 Mathf.PerlinNoise의 예제 스크립트는 다음과 같다.  
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Example1 : MonoBehaviour
{

void Update()
    {
        float height = heightScale * PerlinNoise(Time.time * xScale);
        Vector2 pos = transform.position;
        pos.y = height;
        transform.position = pos;
    }
}
```
간단한 프로그램이며 xScale, heightScale을 통하여 간단하게 진폭과 주기 제어 가능하다.  

지금까지 펄린 노이즈에 대해서 간단히 활용할 수 있는 데모에 대해서 살펴 보았다. 이는 아주 간단한 코드이며 이해하기 어렵지 않지만 펄린 노이즈를 잘 활용하기 위해서는 위 내용들을 인지하여야 할 것이며, 이를 잘 활용한다면 파동 생성, 지형 생성 등 게임에서 랜덤성이 필요한 대부분에 기여할 수 있을 것이다. 추후 펄린 노이즈로 생성한 난수와 행동 트리 바탕으로 게임 속 NPC의 AI의 생성에 관하여 연구하고 업데이트 할 것이다.
