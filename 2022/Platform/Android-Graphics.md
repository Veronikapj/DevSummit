## Android Graphics

### pen / stylus 작동 방법 

- multi buffered rendering : Buffer 2개 -> 시간 소요가 큼

![image-20221218180958604](/Users/kwonhyeokjun/Library/Application Support/typora-user-images/image-20221218180958604.png)

- Front-Buffered Rendering : Buffer 1개 -> 시간 소요 적음
1개의 버퍼에서 모든 역할 수행하며, 이상적인 상태이다. Android는 OpenGL을 사용하여, 모든 버전에서 이러한 구현이 가능합니다.

![image-20221218181033703](/Users/kwonhyeokjun/Library/Application Support/typora-user-images/image-20221218181033703.png)

- OpenGL(graphic lib) : 2차원 또는 3차원 드로잉을 위한 표준 그래픽스 라이브러리.



OpenGL 플랫폼에서 지원을 받지 못하여 새로 만들게 되었다.

Android 13+ 에서는 'Front-Buffered Platform Support' 사용

Front-Buffered Platform Support를 사용함으로써 OpenGL을 통합할 필요가 없게 되었습니다.


추가적으로 'Low-Latency Jetpack library' 적용하여 이전 버전과 호환될 수 있게끔 사용할 수 있습니다. 

Low-Latency Jetpack library

- An easy way to implement stylus-friendly surfaces
- Best possible solution and latency for the version of Android You're running on

![image-20221218190054915](/Users/kwonhyeokjun/Library/Application Support/typora-user-images/image-20221218190054915.png)

Active Stroke Layer에서 그려진 후, 밑에 바닥으로 그려진 것이 옮겨지게 됩니다.



### Blurs

Android 12+

![image-20221218190438882](/Users/kwonhyeokjun/Library/Application Support/typora-user-images/image-20221218190438882.png)


Paint attributes

- common, simple attributes
    - color(색)
    - strokeWidth(두께)
    - style: fill vs stroke (전체 색, 테두리 색)
- More complex attributes
    - android.graphics.ColorFilter
    - android.graphics.Shader(Gradients, bitmaps)

RenderEffect (Android 12+ ) : RenderEffect를 사용하면 더욱 간편하게 그래픽 효과를 적용할 수 있습니다.

- Bundle and chain Shader effects together
- View.setRenderEffect()
    - No need for custom views with shaders
- But wait, there's more!
    - RenderEffects for bitmaps, color, filters, blending
    - ... and Blur

![image-20221218191220279](/Users/kwonhyeokjun/Library/Application Support/typora-user-images/image-20221218191220279.png)

```kotlin
// RenderEffect Blur
// 화면 상단의 seekbar를 통해 화면의 blur(흐릿함) 정도를 조정할 수 있다. 
seekBar.setOnSeekBarChangeListener(object: OnSeekBarChangeListener {
  override fun onProgressChange(
    seekbar: Seekbar, 
    progress: Int, 
    fromUser: Boolean) {
    	updateEffect(progress.toFloat())
  	}
	}
)

fun updateEffect(progress: Float) {
  if(progress > 0) {
    val blur = RenderEffect.createBlurEffect(
    	progress, progress, Shader.TileMode.CLAMP
    )
    pictureGrild.setRenderEffect(blur)
  } else {
    pictureGrid.setRenderEffect(null)
  }
}
```



### SurfaceView vs TextureView

SurfeceView : UI 요소로 계층화된 다른 동영상을 합성하는 경우 성능적 이점을 지닙니다.

- API 1+
- 앱의 view 계층 구조를 통해 표면에 구멍을 뚫습니다. 구멍이 난 곳에 SurfaceView를 넣어주어서, SurfaceView를 볼 수 있게 해줍니다. 
- The surface is assigned a hardware overlay
    - Think of it as a "second, seperate window"
    - Better power efficiency and support for HDR  (High Dynamic Range)

TextureView : content streaming을 보여줄 때 사용하며, 회전 처리에 이점을 갖습니다. 

- API 14+
- Deeper integration into view hierarchy
    - content is drawn to offscreen buffer
    - offscreen buffer is then copied into view hierarchy

We Recommend SurfaceView

- DRM으로 보호된 동영상은 오버레이 창에만 표시할 수 있습니다. 보호된 콘텐츠를 지원하는 동영상 플레이어는 SurfaceView로 구현해야 합니다.
- 게임은 자체 컨텐츠 생산은 SurfaceView을 통해 컨트롤할 수 있습니다.
- TextureView를 사용하는 경우 
    - View-level alpha, rotation, clipping, blend, modes, or RenderEffects에서는 TextureView를 사용하는 것이 좋습니다.
    - 위, 아래에 레이어된 view사이에 존재해야 하는 경우 



## AGSL (Android Graphic Shading Language)

Android 13+  

pixel shader : 최종 색상 값을 계산하기 위해 렌더링 표면의 모든 픽셀에 대해 실행되는 코드

![image-20221218191901912](/Users/kwonhyeokjun/Library/Application Support/typora-user-images/image-20221218191901912.png)

SKSL 사용 예시



AGSL in Android 13+

- AGSL == Android Graphics shading Language
    - Android 에서 사용하기 위한 SKSL (skia shading language)
    - Similar to GLSL ES1.0
- Skia 렌더링 파이프라인에 통합
    - (Skia already uses shaders internalliy)


![image-20221218193616239](/Users/kwonhyeokjun/Library/Application Support/typora-user-images/image-20221218193616239.png)



---

추가 공부

skia : 다양한 하드웨어 및 소프트웨어 플랫폼에서 공통 API를 제공하는 오픈 소스 2D 그래픽 라이브러리입니다. 구글 크롬, 안드로이드, 플러터 등 다양한 제품들의 그래픽 엔진 역할을 합니다.

Low-Latency: 인풋과 아웃풋 간의 지연 간격을 최소화하는 것,

HWUI : Android 3.0 Honeycomb 버전에서부터 공식적으로 태블릿을 위한 OS가 지원되기 시작하였습니다. 이전 SKIA로만 처리되었던 렌더링 방식이, 화면이 커지면서 애니메이션이 부드럽게 작동하지 않는다는 문제점을 마주하였고, 이를 해결하기 위해 HWUI가 도입되었습니다.

Shader : 그래픽 처리 장치(GPU)의 프로그래밍이 가능한 렌더링 파이프라인을 조작할 수 있는 프로그래밍 언어. 렌더링 파이프라인의 최종 목표는 컴퓨터 데이터를 모니터의 픽셀까지 가지고 오는 것이다. Shader를 활용해 색의 농담, 색조, 명암 효과를 줄 수 있습니다.

GLSL(OpenGL Shading Language) : OpenGL에서 동작하는 shader 언어

AGSL을 사용하여 정의된 동작과 함께, 프로그래밍 가능한 <u>RuntimeShader</u> 객체에 대한 지원을 추가합니다. AGSL은 GLSL과 많은 구문을 공유하지만, Android 렌더링 엔진 내에서 작동하여 Android의 캔버스 내에서 그리기를 사용자 설정할 뿐 아니라, View 콘텐츠를 필터링합니다. Android는 내부적으로 이러한 셰이더를 사용하여 잔물결 효과, 흐리기, 스트레치 오버스크롤을 구현하며, android 13을 사용해 유사한 고급 효과를 앱에서 사용할 수 있습니다.

RuntimeShader : 사용자가 정의한 AGSL 함수를 기반으로 하여 픽셀 당 색상을 계산합니다. 