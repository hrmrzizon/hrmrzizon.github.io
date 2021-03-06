---
layout: post
author: "Su-Hyeok Kim"
comments: true
categories:
  - unity
  - shader
  - rendering
  - cg
  - shaderlab
  
---

이전에 쓴 글([handling uvs and material]({{ site.baseurl }}{% post_url 2017-05-15-handling-uv-and-material-in-unity %}))에서 쉐이더에 대한 언급을 한적이 있다. 간단하게 전체적인 의미와 역할에 대해서 설명했었다. 이 글에서는 조금 더 자세하게 알아보고 CG 를 이용해서 직접 다루는 방법에 대해서 알아보겠다.

3D 오브젝트는 GPU 에서 특정한 연산을 하여 화면상에 실제로 그려진다. 예전에는 그리는 방식이 정해져 있어 그 방식에 맞추어 데이터를 넣어주면 GPU 와 Graphics API 가 알아서 3D 오브젝트를 그렸었다. 하지만 기술은 점점 발전하여 프로그래머들이 직접 많은것을 제어할 수 있게 되었고 현재는 꽤 많은 것들이 가능하게 되었다. 그 발전속에서 나타난 것이 쉐이더다. 쉐이더는 3D 오브젝트를 그리는 방식을 적어놓은 코드라고 할 수 있다.

3D 오브젝트를 그리는 쉐이더 코드는 두가지로 나뉘는데, 하나는 vertex 를 처리하는 과정 또 하나는 pixel 자체를 처리하는 코드로 나뉜다. 이 두가지 과정을 잘 처리하면 게임에서 원하는 연출과 성능 두가지 토끼를 잡을 수 있다. 물론 잘하기 힘들다. 그래서 두 방법에서 프로그래머가 직접 코드를 짜서 넣으면서 게임의 그래픽을 원하는대로 커스터마이징이 가능하게 되었다. 이로써 꽤 많은 것을 실현 가능하게 되었었다. 하지만 이게 다가 아니였다.

쉐이더를 사용한 AAA급 3D 게임들과 함께 GPU 도 격렬하게 발전했다. 발전한 만큼 GPU 의 퍼포먼스는 점점 괴물이 되어가고 그 과정에서 vertex shader 와 pixel shader 를 단순하게 그리는 것에만 사용하는 것이 아니라 다른 계산이 필요한 곳에 써먹기 시작했고 편법을 사용한 많은 기술이 나왔었다. ([vtf](http://www.gamedevforever.com/61)) 그렇게 프로그래머의 니즈를 파악한 GPU 제조사는 다른 기술을 개발한다. 이름하여 GPGPU 라는 이름의 기술인데 풀어 쓰면 _"general purpose computing on graphics processing units"_ 이다. _GPU 상의 범용 계산_ 이라는 뜻이다. 즉 위에서 언급한 병렬 계산이 가능한 것들을 편법을 쓰지말고 직접 이 기술을 사용해서 사용하라는 것이다. 이 GPGPU 기술이 나오면서 GPU 의 하드웨어적인 퍼포먼스에 따라 엄청 많은 것들을 가능하게 되었다. GPGPU 를 통해 불편했던 편법을 사용하던 기법들이 변형되어 쏟아져 나왔으며 새로운 기술 또한 엄청나게 쏟아져 나왔다. 그리고 그 기술들은 일반적으로 알려진 3D 그래픽이 차용된 AAA 급 게임들에 사용되어 일반 사용자들은 엄청난 그래픽을 자랑하는 게임들을 경험할 수 있게 되었다. 또한 최근에 _AI_ 기술이 대두되면서 GPGPU 가 더욱더 각광받게 되었다.

이렇게 우리에게 다가오는 것은 꽤 많은 게임들의 발전인데, 다만 우리가 이 게임들의 기술에 접근하려면 꽤 많은 지식과 발상의 전환이 필요하다. 쉐이더만 하더라도 쉐이더 코드는 컴파일되어 GPU 에서 실행된다. CPU 에서 실행되는 일반적인 코드와 조금 다른 점은 CPU 에서 처리되는 것은 멀티스레딩을 하지 않는 이상 상당히 선형적인 코드를 짜게 되고 GPU 에서 돌아가는 쉐이더 코드를 짤 때는 병렬(parallel) 환경에서 돌아가게 짜야한다. 쉐이더 코드를 짤 때 첫번째로 겪게되는 어려움은 이것이다. 쉐이더까지 건드리게되면 경험이 어느정도 있는 상태일텐데, 개념을 조금 깨부수고 아예 병렬적으로 코드를 짜야하니 적응하는 것에 시간이 꽤나 소모된다.

Unity 에서 Shader 를 직접 만들어 사용하는 것에 대하여 알아보자.

<!-- more -->

Unity 는 여러 메인 스트림의 쉐이더 언어를 통해 쉐이더 코딩이 가능하다. 각각 언어마다 큰 차이는 없다. DirectX 와 OpenGL 에서 각각 지원하는 HLSL, GLSL 은 C 기반의 언어이고, Unity 에서 가장 많이 쓰이는 CG 는 NVidia 에서 MS 와 협력하여 만들어졌기 때문에 HLSL 과 비슷할 수 밖에 없다.([Cg & HLSL FAQ](https://web.archive.org/web/20120824051248/http://www.fusionindustries.com/default.asp?page=cg-hlsl-faq)) 또한 쓰이는 문법도 많은 편은 아니라 한가지를 익혀두면 나머지를 사용하는데 크게 불편함은 없을 것이다. 물론 Unity 에서 쓰이는 쉐이더는 ShaderLab 을 기반으로 코딩해야 하기 때문에 네이티브 CG, HLSL, GLSL 과 전체적인 개괄은 다르다. 더 궁금한 사람은 Unity 본사 엔지니어 Aras 가 답변한 [질문 링크](https://forum.unity3d.com/threads/hlsl-cg-shaderlab.4300/) 를 보면 된다.

Unity 의 기본적인 쉐이더 코딩은 ShaderLab 이라는 언어를 사용한다. 아래 ShaderLab 으로 되어있는 예제를 살펴보자.

```
Shader "Custom/TextureColor" {
	Properties {
		_Color ("Color", Color) = (1,1,1,1)
		_MainTex ("Texture", 2D) = "white" {}
	}
	SubShader {
    Tags { "Queue"="Geometry" "RenderType"="Opaque" }

		Pass {
      Lighting Off

			constantColor[_Color]
			SetTexture[_MainTex] { combine texture * constant }
		}
	}

  FallBack "Diffuse"
}
```

위 예제는 색과 텍스쳐를 인자로 받아 텍스쳐에 색을 입혀서 출력해주는 간단한 예제다. 몇 줄 안되는 코드로 텍스쳐와 색을 입혔다. 언어 자체는 단순하고 간결하다. 다만 우리가 알아야하는 몇가지 문법이 있다. CG 나 HLSL 을 사용해도 결국 인라인, 삽입해서 사용하고 기본은 ShdaderLab 이기 때문에 전체를 감싸는 문법은 반드시 알아야 한다.

가장 첫 줄에 Shader 이름을 적어주면 Unity 에서 매터리얼의 쉐이더를 선택하는 부분에 적어준 이름이 나온다. 그리고 밑부분을 보면 Properties 라는 항목들이 있다. 이 부분은 실제로 매터리얼에 저장하는 정보들을 정의해주는 부분으로 지정된 자료형들만 세팅이 가능하다. 위 코드에는 색과 텍스쳐를 넣어줄 수 있게 해놓았다. 그 다음부터는 실제로 렌더링을 하는 부분에 대한 코드들이다. 다만 조금의 구조가 있어 기본적인 사항은 숙지해야 한다. 기본만 알면 쉽게 코딩이 가능하다.

SubShader 는 Shader 안에 여러개가 존재할 수 있는데 이는 꽤나 타당한 이유가 있다. 렌더링은 결국 빛과 여러 색들을 조합해서 화면에 뿌린다. 그리고 GPU 실제로 색을 그려준다. 그런데 낮은 버젼의 GPU 들은 꽤나 지원하지 않는 것들이 많다. GPU 별로 지원하는 Graphics API 버젼이 다른데 최신 기술을 쓰면 낮은 버젼의 Graphics API 를 지원하는 GPU 들은 해당 쉐이더 코드를 실행하지 못한다. 그래서 SubShader 의 개념을 두어 GPU 가 기능을 지원하지 못할 시 코드 상에서 아래 있는 걸로 한계단씩 내려가게 된다. 문제는 모든 SubShader 를 쓰지 못할때다. 그때는 Fallback 키워드에 적혀있는 쉐이더를 사용하여 그리게 한다. 위 예제 코드에서는 Diffuse 쉐이더를 사용하게 했다. 또한 Tag 를 설정해서 SubShader 를 Material 에서 설정할 수도 있다. Standard 쉐이더가 Tag 로 선택하는 기능을 지원한다.

SubShader 는 전체적인 그리는 방법을 포함하는 개념이고 그 다음 하부로 내려가면 Pass 라는 개념이 있다. 이는 진~~~짜로 렌더링을 하는 구문으로써 이 부분에 그리는 방법을 서술한다. CG 나 HLSL 을 넣어줄 수도 있다. 자세한 문법은 [링크](http://chulin28ho.tistory.com/159)를 참조하라.

특별하게 최적화를 할것이 아니라면 ShdaderLab 을 통해서 코딩을 해도 문제가 없다. 다만 좋은 퀄리티의 게임들은 대부분 쉐이더와 여러가지를 최적화를 시켜 주어야 하기 때문에 ShaderLab 만으로는 무리가 있다.

결국 모든 것을 제어하려면 CG 나 HLSL 을 사용해야한다. 그래서 우리는 CG 를 통해서 Unity 에서 쉐이더 코딩을 할 것이다. CG 는 두가지 종류로 쉐이더 코딩을 지원한다. 하나는 표면 쉐이더(surface shader) 를 통한 코딩이고, 하나는 정점 쉐이더(vertex shader) 와 픽셀 쉐이더(pixel shader) 의 조합으로 사용된다.

표면 쉐이더는 실제로는 없는 개념으로 쉐이더를 컴파일하면서 정점/픽셀 쉐이더로 변환되는 쉐이더 기능이다. 보통은 간단하고 빠르게 정점 라이팅을 코딩할 때 쓰인다. 기존에 존재하는 여러 라이팅 모델들을 지원하며 직접 정점 라이팅을 할 수도 있다. 다만 픽셀/프래그먼트 쉐이딩은 안된다. 그래서 상식적으로 생각하면 디퍼드 렌더링에서는 안되겠지만 디퍼드 렌더링에서도 가능하게 만들어 놓았다. 아래 표면 쉐이더의 예제를 보자. Unity 5.5.2f 버젼에서 기본으로 생성되는 쉐이더다.

```
Shader "Custom/NewSurfaceShader" {
	Properties {
		_Color ("Color", Color) = (1,1,1,1)
		_MainTex ("Albedo (RGB)", 2D) = "white" {}
		_Glossiness ("Smoothness", Range(0,1)) = 0.5
		_Metallic ("Metallic", Range(0,1)) = 0.0
	}
	SubShader {
		Tags { "RenderType"="Opaque" }

		CGPROGRAM
		#pragma surface surf Standard fullforwardshadows
    sampler2D _MainTex;

		struct Input {
			float2 uv_MainTex;
		};

		half _Glossiness;
		half _Metallic;
		fixed4 _Color;

		void surf (Input IN, inout SurfaceOutputStandard o) {
			// Albedo comes from a texture tinted by color
			fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
			o.Albedo = c.rgb;
			// Metallic and smoothness come from slider variables
			o.Metallic = _Metallic;
			o.Smoothness = _Glossiness;
			o.Alpha = c.a;
		}
		ENDCG
	}
	FallBack "Diffuse"
}
```

ShdaderLab 에 비하면 코드가 상당히 길다. 새롭게 변수를 세팅해 주어야 하기도 하고 몇가지 세팅을 해주어야 하기 때문이기도 하다. 이 코드에서는 라이팅을 Unity Standard 의 PBR 라이팅 모델을 사용하고 있다. 몇가지 살펴보면, 처음과 마지막을 CGPROGRAM 과 ENDCG 로 감싸준다. 그리고 바로 아래에 C 계열에서 많이 쓰이는 _pragma_ 전처리 키워드를 사용하여 뭔가 정의하고 있는데 바로 표면 쉐이더 함수의 정의를 써주는 곳이다.

```
#pragma surface surf Standard fullforwardshadows
```

surface 는 표면 쉐이더 함수를 정의한다는 것을, surf 는 코드에 정의되어 있는 함수의 이름을, Standard 는 라이팅 모델을 뜻한다. Unity Standard 쉐이더의 PBR 을 모델이다. 그리고 fullforwardshadows 는 그림자에 대한 옵션이다. 메터리얼의 인자에 들어가는 텍스쳐 등 설정값을 변수로 정의해준다. 변수 이름을 위에 Properties 에 정의와 똑같이 써주면 알아서 Unity 에서는 이 값을들을 접근하게 해준다. 반드시 똑같이 써주어야 작동한다.

```
struct Input {
	float2 uv_MainTex;
};
```

사이에 Input 구조체가 있다. 이 구조체에는 단순히 uv_MainTex 라는 변수한개만 있다. 단순하게 이름을 풀이해보면 MainTex 텍스쳐의 uv 좌표를 뜻한다. 이 변수 이름 역시 반드시 맞춰주어야 한다. 앞에는 uv 가 붙어야 하며, 그 다음 존재하는 텍스쳐 이름을 붙여주어야 한다.

```
void surf (Input IN, inout SurfaceOutputStandard o) {
	// Albedo comes from a texture tinted by color
	fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
	o.Albedo = c.rgb;
	// Metallic and smoothness come from slider variables
	o.Metallic = _Metallic;
	o.Smoothness = _Glossiness;
	o.Alpha = c.a;
}
```

이제 남은것은 surf 함수만 남아있다. 이는 표면 쉐이더에서 어떻게 정보를 처리하느냐를 적어주는 곳이다. Input 구조체는 위에서 정의해 주었고, _SurfaceOutputStandard_ 는 Standard 라이팅 모델에서 이미 정의된 구조체이다. 이 구조체의 값을 넣어주는 것으로 쉐이더 연산을 정의한다. 안의 코드를 풀어보자면, 가장 처음에 _tex2D_ 라는 함수로 _\_MainTex_ 라는 텍스쳐와 _IN.uv\_MainTex_ uv 좌표를 이용해 색을 가져온다. 그리고 _\_Color_ 변수를 가져온 색에 각각에 rgba 별로 곱연산을 해준다. 그렇게 계산된 fixed4 자료형 _c_ 의 데이터를 스탠다드 라이팅 모델 인자의 Albedo 변수에는 rgb 값을, Alpha 변수에는 a 값을 넘겨준다. 나머지는 입력된 데이터 그대로 넣어준다.

간단하게 표면 쉐이더 코드에 대해서 살펴보았다. 한가지 짚고 넘어가야 될것은 표면 쉐이더는 Unity 에서 자동으로 원래의 개념으로 컴파일 해주는 일종의  위 예제에서 본 Standard 라이팅 모델 말고도 미리 정의되어 있는 다른 라이팅 모델을 사용할 수도 있고 직접 라이팅 모델을 정의해서 사용할 수도 있다. 이 글에서는 라이팅 모델에 대해서 자세한 것은 다루지 않겠다. 자세한 사항에 대해서는 [Unity 표면 쉐이더 레퍼런스](https://docs.unity3d.com/kr/current/Manual/SL-SurfaceShaders.html) 를 참조하라.

다음으로 살펴볼 쉐이더는 정점/픽셀 쉐이더를 조합한 쉐이더다. CG 를 사용하는 가장 낮은 단계의 쉐이더이며 실질적으로 돌아가는 쉐이더다. 그만큼 해야할 것도 많고 신경써야 할것도 많다. 특히 라이팅을 세팅할 때는 꽤나 코드가 길어지고 복잡해진다. 하지만 그만큼 세세하게 조정이 가능하다. 그래서 보통은 특수효과에 많이 쓰이며 최적화를 위한 쉐이딩을 할때도 쓰인다. 또한 여러 다른 쉐이더를 사용할 때에도 쓰인다. 아래 예제를 살펴보자.

```
Shader "Custom/ColorTextureCG" {
	Properties {
		_Color ("Color", Color) = (1,1,1,1)
		_MainTex ("Albedo (RGB)", 2D) = "white" {}
	}
	SubShader {
		Tags { "RenderType"="Opaque" }

		Pass {
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag

			#include "UnityCG.cginc"

			struct appdata
			{
				float4 vertex : POSITION;
				float2 uv : TEXCOORD0;
			};

			struct v2f
			{
				float4 vertex : SV_POSITION;
        float2 uv : TEXCOORD0;
			};

			v2f vert (appdata v)
			{
				v2f o;
				o.vertex = mul(UNITY_MATRIX_MVP, v.vertex);
				o.uv = v.uv;
				return o;
			}

			fixed4 _Color;
			sampler2D _MainTex;

			fixed4 frag (v2f i) : SV_Target
			{
				fixed4 col = tex2D(_MainTex, i.uv);
				return col * _Color;
			}
			ENDCG
		}
	}

	FallBack "Diffuse"
}
```

위 예제는 색과 텍스쳐를 입혀주는 쉐이더로 아주 간단한 코드로 이루어져 있다. 한번 살펴보자. 그리고 CG 에서는 픽셀 쉐이더의 픽셀을 _fragment_ 라고 칭한다.

역시나 CGPROGRAM 과 ENDCG 로 감싸져 있으며 가장 처음에는 _pragma_ 구문이 등장한다. 표면 쉐이더에서는 함수와 라이팅 모델을 설정하는데 쓰였는데, 여기서도 정점/픽셀 쉐이더 함수를 설정해주는데 쓰인다. 정점 쉐이더는 vertex 오른쪽에 함수 이름을 넘겨주고, 픽셀 쉐이더는 fragment 오른쪽에 함수 이름을 넘겨준다. 말 그리고 표면 쉐이더가 내부의 동작 원리를 알기 힘든것과는 다르게 정점/픽셀 쉐이더는 보이는 코드와 같이 똑같이 동작한다. 정점/픽셀 별로 처리하는 과정이 다르며 예제에 써준 코드 그대로 실행된다는 뜻이다.

_pragma_ 구문 아래에 표면 쉐이더 코드에서는 못보던 것이 있다. 바로 C 프로그래밍을 했을 떄 보던 _include_ 키워드다. 우리가 보는것과 같이 해당 위치에 _UnityCG.cginc_ 라는 이름을 가진 파일의 내용을 넣어주는 역할을 하는데, 이 _UnityCG.cginc_ 라는 파일에는 Unity 에서 제공하는 파일로 Unity 상에서 필요한 데이터들을 접근하고 데이터을 변환해주는 함수들이 들어있다. 정점이 _UnityCG.cginc_ 파일은 필수적으로 넣어주어야 한다.

이제 직접 정점 쉐이더와 픽셀 쉐이더를 계산하는 부분들이 남았다. _vert_ 함수를 보면 _appdata_ 라는 구조체 변수를 인자로 받아서 _v2f_ 구조체 데이터를 반환하는 간단한 함수로 보인다. 이 함수의 코드를 보자.

```
v2f vert (appdata v)
{
	v2f o;
	o.vertex = mul(UNITY_MATRIX_MVP, v.vertex);
	o.uv = v.uv;
	return o;
}
```

첫번째 줄은 실제 Unity 의 위치와 _Mesh_ 에 저장되어 있던 로컬 정점 위치를 계산하여 정점의 실제 위치를 계산하여 넣어주는 줄이다. 실제로는 행렬 변환을 통해 위치값을 변경한다. 이 줄을 빼버리면 (0,0,0) 을 기준으로 모델이 출력될 것이다. 두번째 줄에서는 UV 좌표값을 그대로 넣어준다. 이 UV 좌표는 픽셀 쉐이더에서 사용한다.

내부적으로 복잡한 처리가 끝나고 다음으로 픽셀 쉐이더 단계로 넘어간다. 등록된 함수가 호출되며 위의 _frag_ 함수가 호출되는데 여기서 텍스쳐와 UV 좌표로 폴리곤에 색을 입혀준다. 내용을 보자.

```
fixed4 frag (v2f i) : SV_Target
{
	fixed4 col = tex2D(_MainTex, i.uv);
	return col * _Color;
}
```

_frag_ 함수의 타입이 지정된 첫줄에 마지막에 쓰여있는 _SV\_TARGET_ 은 어떤 곳에 픽셀 쉐이딩의 결과를 저장하는지에 대한 선언이다. Render Target 이라고 한다. _SV\_TARGET_ 을 기본으로 쓰는데 이는 한곳에 모든 데이터를 쓴다는 선언이다. 여러곳에 데이터를 기록하면 Multiple Render Target 이라고 칭하게 되는 기술을 쓰는 것이다. 문법에 더 궁금한 사람은 [Semantics](https://docs.unity3d.com/Manual/SL-ShaderSemantics.html) 를 보라. MRT 에 대해 궁금한 사람은 [Wiki: MRT](https://en.wikipedia.org/wiki/Multiple_Render_Targets) 를 보라.

이제 내용을 보면 tex2D 함수에 _\_MainTex_ 텍스쳐와 _i.uv_ 데이터를 넣어주어 추출된 색을 _col_ 변수에 저장한다. 이 안에는 RGBA 데이터가 들어있다. 그리고 _col_ 변수와 _\_Color_ 변수를 _col_ 변수에 곱하여 실제 출력되는 픽셀 색을 반환한다. 반환된 색은 이제 실제로 보이게 된다.

자세한 사항은 [Unity 정점/픽셀 쉐이더 레퍼런스](https://docs.unity3d.com/kr/current/Manual/SL-ShaderPrograms.html) 를 참조하면 된다.

여기까지 텍스쳐와 색을 입히는 정점/픽셀 쉐이더에 대해서 알아보았다. 아주 기본적인 것들만 다루었기에 코드는 단순하다. 여기에 여러 효과들과 여러 기법들이 많이 들어가면 들어갈수록 복잡해질 것이다. 게다가 여러 플랫폼을 지원하기 위해 여러개의 SubShader 와 Pass 를 넣게되면 엄~청 긴 코드가 나올 것이다.

그리고 번외로 Unity 컴포넌트 안에서 픽셀을 직접 만져서 바꿀 수 있는 기능이 있다. [링크](https://docs.unity3d.com/kr/current/ScriptReference/MonoBehaviour.OnRenderImage.html)를 참조하라. 이 기능을 통해 Standard Asset 에 꽤 많은 효과들을 지원하는 기능들이 있다.

## 참조

 - [Khronos : Fixed function pipeline](https://www.khronos.org/opengl/wiki/Fixed_Function_Pipeline)
 - [Unity ref : shader references](https://docs.unity3d.com/kr/current/Manual/SL-Reference.html)
 - [Unity forum : hlsl? cg? shaderlab?](https://forum.unity3d.com/threads/hlsl-cg-shaderlab.4300/)
 - [Unity forum : CG Toolkit is legacy](https://forum.unity3d.com/threads/cg-toolkit-legacy.238181/)
 - [NVidia developer : CG Toolkit](https://developer.nvidia.com/cg-toolkit)
 - [Unity tutorial : Shader tutorial](https://unity3d.com/kr/learn/tutorials/topics/graphics/gentle-introduction-shaders)
 - [블로그 : Shaderlab Ref](http://chulin28ho.tistory.com/159)
 - [블로그 : 세이더 기초](http://jinhomang.tistory.com/43)
 - [Unity wikibooks : cg programming](https://en.wikibooks.org/wiki/Cg_Programming/Unity)
