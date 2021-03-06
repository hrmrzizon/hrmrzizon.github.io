---
layout: post
author: "Su-Hyeok Kim"
comments: true
categories:
  - opengles
  - optimization
---

※ [docs.nvidia : OpenGL ES Programming Tips](https://docs.nvidia.com/drive/drive_os_5.1.6.1L/nvvib_docs/index.html#page/DRIVE_OS_Linux_SDK_Development_Guide/Graphics/graphics_opengl.html)의 내용을 번역하는 포스팅입니다. 시간이 꽤 지난 내용으로 글의 내용은 현재 상황과 다를 수 있습니다. 또한 모든 내용이 번역되어 있지 않을 수도 있습니다.

<!-- more -->

This topic is for readers who have some experience programming OpenGL ES and want to improve the performance of their OpenGL ES application. It aims at providing recommendations on getting the most out of the API and hardware resources without diving into too many architectural details.

해당 주제는 OpenGL ES 의 익숙한 경험을 가지고 있고 OpenGL ES 응용프로그램의 성능을 향상시키고 싶어하는 독자를 위해 작성되었습니다. 해당 문서는 아키텍쳐적인 디테일에 너무 들어가지 않고 API와 하드웨어 자원을 최대한 활용하는 방법을 제공하는 것에 초점이 맞춰져 있습니다.

# 프로그래밍 효율

Some of the recommendations in this topic are incompatible with each other. One must consider the trade-offs between CPU load, memory, bandwidth, shader processing power, precision, and image quality. Premature assumptions about the effectiveness of trade-offs should be avoided. The only definite answers come from benchmarking and looking at rendered images!

이 토픽의 몇몇 사항들은 서로 호환되지 않습니다. 반드시 CPU 부하, 메모리, 대역폭, 쉐이더 처리 능력, 정확도, 이미지 퀄리티 사이의 trade-off 를 고려해야 합니다. trade-off의 효율에 대하여 섣부른 가정은 피해야 합니다. 오직 결정론적인 답은 벤치마킹과 렌더링된 이미지를 봄으로써 알 수 있습니다. 

The items below are not ordered according to importance or potential performance increase. The identifiers in parentheses exist only for reference.

아래의 항목은 중요성 혹은 잠재적인 성능 향상에 대해 정렬되지 않았습니다. 레퍼런스를 위해 괄호안의 식별자만 제공됩니다.

# 상태

Inefficient management of GL state leads to increased CPU load that may limit the amount of useful work the CPU could be doing elsewhere. Reducing the number of times rendering is paused due to GL state change will increase the chance of realizing the potential throughput of the GPU. The main point in this section is: do not modify or query GL state unless absolutely necessarily.

_GL state_ 의 비효율적인 관리는 CPU 부하를 증가시키고, 이는 어디서든 CPU의 유용한 일을 제한시키게 될 것입니다. _GL State_ 의 상태 변경을 위한 렌더링이 일시 중지되는 경우를 줄이는 것은 GPU의 잠재적인 처리량을 실현시킬 기회를 늘릴 수 있습니다. 이 섹션에서 중요한 점은, 완전히 필요한 경우가 아니라면 _GL state_ 를 바꾸거나 query하지 않는 것입니다.

## 쓸모없이 _GL State_ 를 변경하지 마십시오. (S1)

All relevant GL state should be initialized during application initialization and not in the main render loop. For instance, occasionally glClearDepthf, glColorMask, or glViewport finds its way into the application render loop even though the values passed to these functions are always constant. Other times they are set unconditionally in the loop, just in case their values have changed per frame. Only call these functions when the values actually do need to change. Additionally, do not automatically set state back to some predefined value (e.g., the GL defaults). That idiom might be useful while developing your application as it makes it easier to re-order pieces of rendering code, but it should not be done in production code without a very good reason.

모든 관련된 _GL state_ 는 메인 렌더링 루프가 아닌 응용프로그램의 초기화 동안 초기화 되어야 합니다. 예를 들면, _glClearDepthf, glColorMask, glViewport_ 는 전달되는 값이 항상 상수이더라도, 렌더링 루프에 들어갑니다. 다른 때에 프레임별로 값이 달라지는 케이스만 고려하여 루프에서 조건없이 계속 세팅됩니다. 값이 진짜로 바뀔 때, 함수를 호출하십시오. 추가적으로, GL에 의해 미리 제공된 값으로 자동으로 _GL state_ 를 돌리지 마세요. 이는 당신의 응용 프로그램을 개발할 때 렌더링 코드 조각을 재정렬하기 쉽게 만들어주므로 유용하지만, 아주 중요한 이유 없이는 제품의 코드에 들어가서는 안됩니다.

## 렌더 루프에서 _GL state_ 쿼리를 피하십시오 (S2)

When a GL context is created, the state is initially well-defined. Every state variable has a default value that is described in the OpenGL ES specification ("State Tables"). Except when compiling shaders, determining available extensions, or the application needs to query implementation specific constants, there should be no need to query any GL state. These queries can almost always be done in initialization. Well-written applications check for GL errors in debug builds. If no errors are reported as a result of changing state, it is assumed that the changes are now part of the new GL state. For these two reasons, the current state is always known and you should almost never need to query any GL state in a loop. If an application frequently calls functions that begin with glIs* or glGet*, these calls should be tracked down and eliminated.

_GL context_ 가 생성되었을 때, _GL state_ 는 초기에 잘 정의되어 있습니다. 모든 _GL state_ 의 변수는 OpenGL ES 스펙에("State Tables") 묘사되어 있는 기본 값으로 설정되어 있습니다. 쉐이더를 컴파일 할 때, 가능한 확장을 결정하는 때, 응용 프로그램이 특정 상수를 가져오는 구현이 필요할 때를 제외하면 _GL state_ 를 쿼리할 필요는 없습뇌다. 이 쿼리들은 초기화 과정에서 거의 끝나있기 때문 입니다. 잘 정의된 응용 프로그램은 디버그 빌드 시에 GL 에러를 체크합니다. 이러한 두가지 이유 덕분에 현재 상태는 언제나 알고 있고, 루프에서 GL 상태를 쿼리할 필요가 '절대' 없습니다. 만약 응용 프로그램이 glIs*, glGet* 같은 함수를 빈번하게 호출한다면, 추적하고 제거해야 합니다.

## 공유된 상태를 한번에 실행하십시오. (S3)

An efficient approach to reduce the number of state changes is batching together all draw calls that use the same state (shaders, blending, textures, etc.). For instance, not batching on the shader changes has the form:
_GL state_ 의 상태 변경을 줄이는 효과적인 접근 방법은 같은 상태를 공유하는(쉐이더, 블렌딩모드, 텍스쳐, 등..) _drawwcall_ 을 한번에 합쳐서 실행하는(_batch_) 것입니다. 예를 들면 쉐이더 변화를 배칭하지 않은 것은 아래와 같은 형태를 띕니다:

```
[ UseProgram(21), DrawX1, UseProgram(59), DrawY1, UseProgram(21), DrawX2, UseProgram(59), DrawY2 ]
```

Batching on the shaders leads to an improvement (fewer shader changes):

쉐이더로 배칭하면 다음과 같이 개선시킬 수 있습니다.(적은 쉐이더 변경):

```
[ UseProgram(21), DrawX1, DrawX2, UseProgram(59), DrawY1, DrawY2 ]

```

It is quite effective to group draw calls based on the shader programs they use. Reprogramming the GPU by switching between shaders is a relatively costly operation. Within each batch, a further optimization can be made by grouping draw calls that use the same uniforms, texture objects and other state. Generating a log of all function calls into the OpenGL ES API is a good approach for revealing poor batching. A tool such as PerfHUDES can conveniently generate this log without rebuilding the GL application; no change to the source code is necessary.

사용하는 쉐이더 프로그램을 기반으로 _drawcall_ 을 모으는 것은 꽤 효과적입니다. 쉐이더에 따른 변경으로 GPU를 재-프로그래밍 하는 것은 상대적으로 값 비싼 연산입니다. 같은 uniform, 같은 텍스쳐 오브젝트, 이외의 같은 상태를 가진 것끼리 묶는 각각의 _batch_ 를 통하여 더 나은 최적화를 할 수 있습니다. PerfHUDES 같은 도구는 GL 응용 프로그램을 다시 만드는 것 없이 편리하게 이런 로그를 만들 수 있어 소스 코드를 변경할 필요가 없습니다.

## 바인딩 할 때 오브젝트 별 상태를 반복하지 마십시오. (S4)

Recall that some state is bound to the object it affects. As that state is stored in the object, you do not need to repeat it when you rebind the object. A very common mistake is setting the texture parameters for filtering and wrapping every time a texture object is bound. Another common mistake is updating uniform variables that have not changed value since the last time the particular shader program was used. In particular, when batching opportunities are limited, repeating per object state generates enormously inefficient GL code that can easily have a measurable impact on framerate.

다시 말하자면, 어떤 상태는 영향을 받는 오브젝트에 바인딩 되어 있습니다. 그 상태는 오브젝트의 상태에 저장되어 있으므로, 다시 오브젝트를 바인딩 할때, 이를 반복할 필요가 없습니다. 가장 일반적인 실수는 텍스쳐 오브젝트를 바인딩 할 때 모든 경우에 필터링과 래핑 텍스쳐 파라미터를 설정하는 것입니다. 다른 일반적인 실수는 특정 쉐이더가 마지막으로 사용되었고 값이 바뀌지 않았을 때, uniform 변수에 업데이트 하는 것입니다. 특히 배칭할 기회가 제한되어 있을 때, 오브젝트 상태를 반복하는 것은 프레임레이트에 관측가능한 임팩트를 쉽게 측정 가능한 방대한 비효율적인 GL 코드를 생성합니다.

## backface culling 은 가능하면 켜십시오. (S5)

Always enable face culling whenever the back-faces are not visible. Rendering the back-faces of primitives is often not necessary.

표면의 뒷면이 안보일 때는 언제나 _face culling_ 을 켜십시오. 프리미티브의 뒷면을 그리는건 거의 필요 없습니다.

Note: The default GL state has backface culling disabled, so this is one state that should almost always be set during application initialization and be left enabled for the application lifetime.

참고: 기본적인 _GL state_ 는 _backface culling_ 이 꺼져 있기에, 응용 프로그램을 초기화 할때 거의 대부분 한 상태에 이를 적용시켜야 하고 응용 프로그램이 켜진 동안에 항상 켜져 있어야 합니다.

# 기하

The amount of geometry, as well as the way it is transferred to the GL, can have a very large impact on both CPU and GPU load. If the application sends geometry inefficiently or too frequently, that alone can create a bottleneck on the CPU side that does not give the GPU enough data to work efficiently. Geometry must be submitted in sizable chunks to realize the potential of the GPU. At the same time, geometry should be minimally defined, stored on the server side, and it should be accessed in a way to get the most out of the two GPU caches that exist before and after vertices are transformed.

기하의 양과 GL로 전송되는 방법은 CPU/GPU 부하에 아주 많은 영향을 끼친다. 만약에 응용프로그램이 기하를 비효율적으로 혹은 너무 자주 보낸다면, 그것 만으로도 CPU 쪽에서 GPU가 충분히 효율적으로 일하게에 불충분하게 전송하는 병목이 생길 수 있다. 기하는 반드시 GPU의 잠재적인 성능을 현실화하기 위해 큰 덩어리로 보내져야 합니다. 동시에 기하는 최소한으로 정의되어야 하며, 서버에 저장되어야 하고, 정점이 변환된 전후에 존재하는 두 GPU 캐시를 최대한 활용하는 방법으로 엑세스 되어야 합니다.

## 인덱스된 프리미티브를 이용하십시오. (G1)

The vertex processing engine contains a cache where previously transformed vertices are stored. It is called Post-TnL vertex cache. Taking full advantage of this cache can lead to very large performance improvement when vertex processing is the bottleneck. To fully utilize it, it is necessary for the GPU to be able to recognize previously transformed vertices. This can only be accomplished by specifying indexed primitives for the geometry. However, for any non-trivial geometry the optimal order of indices will not be obvious to the programmer. If some geometry is complex, and the application bottleneck is vertex processing, then look into computing a vertex order that maximizes the hit ratio of the Post TnL cache. The topic has been thoroughly studied for years and even simple greedy algorithms can provide a substantial performance boost. Good results have been reported with the algorithm described at the below locations.

정점 처리 엔진은 이전에 변환된 정점을 저장하는 캐시를 가지고 있습니다. 이는 Post-T&L 정점 캐시로 불립니다. 이 캐시를 최대한으로 이용하는 것은 버텍스 처리에 병목이 있을 때 매우 큰 성능 향상을 이뤄낼 수 있습니다. 최대한 사용하기 위해, GPU에서는 이전에 변환된 정점을 확인하는 것이 필요합니다. 이는 기하를 위해 인덱싱된 프리미티브를 명시하는 것으로 이뤄질 수 있습니다. 하지만 어떤 중요한 가히를 위해 인덱스의 최적화된 순서는 프로그래머에겐 중요하지 않을 수 있습니다. 만약 어떤 기하가 복잡하고 응용 프로그램이 정점 처리에 병목이 있다면 Post T&L 캐시의 히트율을 최대화 하기 위해 정점의 순서를 살펴보세요. 이 토픽은 몇년간 전체적으로 연구되었고, 심지어 간단한 그리디 알고리즘으로 실질적인 성능 향상을 제공합니다. 아래 언급된 알고리즘은 좋은 결과를 보고한 적 있습니다.

| document | URL to latest |
| ----- | ----- |
| Linear-Speed Vertex Cache Optimisation, by Tom Forsyth, RAD Game Tools (28th September 2006) | [URL](https://tomforsyth1000.github.io/papers/fast_vert_cache_opt.html) |

There is a free implementation of the algorithm in a library called vcacne.

이는 vcacne 이라는 라이브러리에 무료로 제공되어 있습니다.

Note: The number of vertex attributes and the size of each attribute may determine the efficiency of this cache—it has storage for a fixed number of bytes or active attributes, not a fixed number of vertices. A lot of attribute data per vertex increases the risk of cache misses, resulting in potentially redundant transformations of the same vertices. 

참고:  많은 버텍스 어트리뷰트와 각각의 어트리뷰트의 크기는 이 캐시의 효율성을 결정할 것입니다―이는 고정된 수의 정점의 갯수가 아닌 고정된 바이트 혹은 활성된 어트리뷰트를 가지고 있기 때문입니다. 버텍스별 많은 어트리뷰트 데이터는 캐시 미스의 리스크를 증가시키며, 잠재적으로 같은 정점의 불필요한 변환을 일으킵니다. 

## 버텍스 어트리뷰트의 크기와 그 갯수를 줄이십시오. (G2)

It is important to use an appropriate attribute size and minimize the number of components to avoid wasting memory bandwidth and to increase the efficiency of the cache that stores pre-transformed vertices. This cache is called Pre-TnL vertex cache. For instance, you rarely need to specify attributes in 32 bit FLOATs. It might be possible to define the object-space geometry using 3 BYTEs per vertex for a simple object, or 3 SHORTs for a more complex or larger object. If the geometry requires floating-point representation, half-floats (available in extension OES_vertex_half_float.txt) may be sufficient. Per vertex colors are accurately stored with 3 x BYTEs with a flag to normalize in VertexAttributePointer. Texture coordinates can sometimes be represented with BYTEs or SHORTs with a flag to normalize (if not tiling).

메모리 대역폭을 낭비하는 것을 피하고, 변환 이전의 정점을 저장하기 위한 캐시의 효율을 높이기 위해 적절한 어트리뷰트 사이즈의 사용 및 그 수를 줄이는 것은 중요한 작업입니다. 이 캐시는 Pre T&L 정점 캐시라고 불리는데, 예를 들어, 32비트 부동소수점을 사용하는 일은 거의 없습니다. 보통 오브젝트 공간 기하를 간단한 오브젝트를 정점별로 3개의 BYTE를 사용하거나 복잡한 오브젝트의 경우 3개의 SHORT를 사용합니다. 만약에 기하가 부동소수점 표현이 필요하다면 half-precision 부동소수점이면 충분할 것입니다.(available in extension _OES_vertex_half_float.txt_) 정점별 색은 정확하게 정규화하기 위한 _VertexAttributepointer_ 플래그와 함께 3개의 BYTE에 저장합니다. 텍스쳐 좌표는 종종 정규화하기 위한 플래그와 함께 몇개의 BYTE나 몇개의 SHORT로 표현됩니다. (타일링 하지 않는 경우)

Note: The exception case that normalizing texture coordinates is not necessary if they are only used to sample a cube map texture.
참고: 예외적인 케이스는 큐브맵 텍스쳐를 샘플링 할 때만 텍스쳐 좌표가 정규화가 필요 없는 경우입니다.

Vertex normals can often be represented with 3 SHORTs (in a few cases, such as for cuboids, even as 3 BYTEs) and these should be normalized. Normals can even be represented with 2 components if the direction (sign) of the normal is implicit, given its length is known to be 1. The remaining coordinate can be derived in a vertex shader (e.g. z = SQRT(1 - x * x + y * y)) if memory or bandwidth (rather than vertex processing) is a likely bottleneck.

정점 법선은 종종 3개의 SHORT 표현되고 (몇개의 경우, cuboid의 경우, 3 BYTE로 충분합니다.) 정규화 되어야 합니다. 심지어 법선은 방향인(부호화된) 경우 길이가 1이므로 두 컴포넌트로 표현될 수 있습니다. 만약 메모리나 대역폭이 병목인 경우 남은 컴포넌트는 버텍스 쉐이더에서 계산될 수 있습니다.

An optimal OpenGL ES application will take advantage of any characteristics specific to the geometry. For instance, a smooth sphere uses the normalized vertex coordinates as normal—these are trivially computed in a vertex shader. It is important to benchmark intermediate results to ensure the vertex processing engine is not already saturated. Finally remember, if some attribute for a primitive or a number of primitives is constant for the same draw call, then disable the particular vertex attribute index and set the constant value with _VertexAttrib_ instead of replicating the data.

최적의 OpenGL ES 응용 프로그램은 기하에 대한 모든 세부적인 특성을 이용합니다. 예를 들어 매끄러운 구는 정규화된 정점 위치를 법선으로 사용 가능하고―이는 보통 정점 쉐이더에서 계산됩니다. 정점 처리 엔진이 아직 과부하 되지 않은 경우를 보장하가 위해 중간 결과를 벤치마크 하는 것은 중요합니다. 마지막으로 기억할 것은 어떤 혹은 다수의 프리미티브를 위한 어트리뷰트가 같은 _drawcall_ 에서 상수라면, 그 정점 어트리뷰트 인덱스를 비활성화 하고 데이터를 복사하는 대신 _VertexAttrib_ 와 함께 상수 값으로 설정하는 것입니다.

## 정점 어트리뷰트를 압축하십시오. (G3)

Vertex attributes normally have different sets of attributes that are completely unrelated. Unlike uniform and varying variables in shader programs, vertex attributes do not get automatically packed, and the number of vertex attributes is a limited resource. Failure to pack these attributes together may lead to limitations sooner than expected. It is more efficient to pack the components into fewer attributes even though they may not be logically related. For instance, if each vertex comes with two sets of texture coordinates for multi-texturing, these can often be combined these into one attribute with four components instead of two attributes with two components. Unpacking and swizzling components is rarely a performance consideration.

정점 어트리뷰트는 보통 서로 완전히 상관없는 다른 어트리뷰트 집합을 가집니다. 쉐이더 프로그램 내의 _uniform_ 과 _varying_ 변수와는 다르게, 정점 어트리뷰트는 자동으로 압축되어지지 않고, 버텍스 어트리뷰트의 크기는 제한된 자원입니다. 이 어트리뷰트들을 압축하는 것에 실패는 예상한 바와 다르게 빠르게 제한이 발생할 수 있습니다. 논리적으로 관련되지 않은 것이라도, 컴포넌트들을 압축하여 적은 어트리뷰트를 사용하는 것은 더 효율적입니다. 예를 들어, 각각의 정점에 여러 텍스쳐를 사용하기 위해 두개의 텍스쳐 좌표를 사용한다고 가정한다면, 두 컴포넌트를 위해 두 어트리뷰트를 사용하는 것 보다, 하나의 어트리뷰트에 4개의 컴포넌트를 결합하는 경우가 종종 있습니다. 압축을 풀고, 컴포넌트를 뒤섞는 것은 대부분 고려되지 않습니다.

## 적절한 정점 어트리뷰트 레이아웃을 선택하세요. (G4)

There are two commonly used ways of storing vertex attributes:

- Array of structures
- Structures of arrays

An __array of structures__ stores the attributes for a given vertex sequentially with an appropriate offset for each attribute and a non-zero stride. The stride is computed from the number of attribute components and their sizes. An array of structures is the preferred way of storing vertex attributes due to more efficient memory access. If the vertex attributes are constant (not updated in the render loop) there is no question that an array of structures is the preferred layout.

보통 두 방법으로 정점 어트리뷰트를 저장합니다.

- 구조체의 배열
- 배열의 구조체

__구조체의 배열__ 은 주어진 정점을 각각의 어트리뷰트에 적절한 오프셋과 _non-zero stride_ 와 함께 주어진 정점을 순차적으로 저장합니다. 이 _stride_ 는 어트리뷰트의 갯수와 각각의 크기에 따라 계산됩니다. __구조체의 배열__ 은 정점 어트리뷰트를 저장하는 더 효율적인 메모리 접근으로 선호되는 방법입니다. 만약에 정점 어트리뷰트가 상수라면(렌더 루프에서 업데이트 되지 않는다면) 당연히 __구조체의 배열__ 은 선호되는 레이아웃 입니다.

In contrast, a __structure of arrays__ stores the vertex attributes in separate buffers using the same offset for each attribute and a stride of zero. This layout forces the GPU to jump around and fetch from different memory locations as it assembles the needed attributes for each vertex. The structure of arrays layout is therefore less efficient than an array of structures in most cases. The only time to consider a structure of arrays layout is if one or more attributes must be updated dynamically. Strided writes in array of structures can be expensive relative to the number of bytes modified. In this scenario, the recommendation is to partition the attributes such that constant and dynamic attributes can be read and written sequentially, respectively. The attributes that remain constant should be stored in an array of structures. The attributes that are updated dynamically should be stored in smaller separate buffer objects (or perhaps just a single buffer if the attributes are updated with the same frequency).

반면에 __배열의 구조체__ 는 정점 어트리뷰트를 같은 어트리뷰트 오프셋과 _zero stride_ 으로 각각의 다른 버퍼에 저장합니디ㅏ. 이 레이아웃은 필요한 정점별 어트리뷰트를 조합하기 위해 GPU가 다른 메모리 위치에 점프하여 가져오게 만듭니다. __배열의 구조체__ 레이아웃은 그러므로 __구조체의 배열__ 보다 대부분의 경우에 비효율적입니다. 오직 __배열의 구조체__ 를 고려해야할 때는 하나 혹은 여러개의 어트리뷰트가 동적으로 업데이트 될 때 입니다. __배열의 구조체__ 에서 _strided_ 쓰기는 얼마만큼의 바이트가 수정된 만큼 비싸질 수 있습니다. 이 시나리오에서 추천할 방법은, 순차적으로 읽기/쓰기가 가능하도록 동적인 어트리뷰트와 정적인 어트리뷰트를 나누는 것입니다. 정적으로 남은 어트리뷰트는 __구조체의 배열__에 저장합니다. 동적으로 업데이트되는 어트리뷰트는 비교적 적인 분리된 버퍼 오브젝트에 저장되어야 합니다.(혹은 같은 주기로 업데이트 될 때 같은 버퍼에 저장합니다.)

## 일관적인 시계/반시계 순서를 사용하십시오. (G5)

The geometry winding (clockwise or counter-clockwise) should be determined up front and defined in code. The geometry face that is culled by GL can be changed with the FrontFace function, but having to switch back and forth between winding for different geometry batches during rendering is not optimal for performance and can be avoided in most cases.

기하 감기(시계 혹은 반시계 방향)는 반드시 미리 코드에서 정의되어야 합니다. GL에서 컬링된 기하 표면은 FrontFace 함수를 통하여 변경될 수 있지만 렌더링 중에 다른 기하 _batch_ 를 위해 앞뒤를 바꾸는 것은 성능에 최적이 아니며, 대부분의 경우 피할 수 있습니다.

## 항상 버텍스, 인덱스 버퍼를 사용하십시오. (G6)

Recall that vertices for geometry can either be sourced from application memory every time it is rendered or from buffers in graphics memory where it has been stored previously. The same applies to vertex array indices. To achieve good performance, you should never continuously source the data from application memory with DrawArrays. Buffer objects should always be used to store both geometry and indices. Check that no code is calling DrawArrays, and that no code is calling DrawElements without a buffer bind.
The default buffer usage flag when allocating buffer objects is __STATIC_DRAW__. In many cases this will lead to fastest access.
Note: __STATIC_DRAW__ does not mean one can never write to the buffer (although any writing to a buffer should always be avoided as much as possible). A __STATIC_DRAW__ flag may in fact be the appropriate usage flags, even if the buffer contents are updated every few frames. Only after careful benchmarking and arriving at conclusive results should changing the usage flag to one of the alternatives (__DYNAMIC_DRAW__ or __STREAM_DRAW__) be considered.

기하를 위한 정점들은 응용 프로그램 메모리에서 어느 때나 렌더링하기 위해, 그래픽 메모리에서 이전에 저장된 것을 위해 참조됩니다. 이는 인덱스 버퍼에도 적용됩니다. 좋은 성능을 얻기 위해서, 당신은 DrawArray를 사용하여 응용 프로그램 메모리에서 연속적으로 참조하면 절대 안됩니다. 버퍼 오브젝트는 언제나 기하와 인덱스를 저장하기위해 사용되어야 합니다. DrawArrays를 사용하는 코드가 없는지 확인하고, 버퍼 바인딩 없이 DrawElements를 호출하는 코드가 없는지 확인하세요. 할당 시 버퍼 오브젝트의 디폴트 버퍼 사용 플래그는 __STATIC_DRAW__ 입니다. 

참고 : __STATIC_DRAW__ 는 버퍼에 한번도 쓰지 않을 수 있다는 것을 의미하지 않습니다.(모든 버퍼에 쓰기는 최대한 피해야 함에도 불구하고) 심지어 버퍼의 내용이 모든 몇 프레임에 업데이트 되더라도, __STATIC_DRAW__ 플래그는 적절한 사용 플래그가 될 것입니다. 오직 조심스러운 벤치마킹과 이의 결과를 통한 추론이 고려된 여러 대안 중의 하나로 바꾸어야 합니다.(__DYNAMIC_DRAW__ or __STREAM_DRAW__)

## 기하를 적은 버퍼와 드로우콜로 묶으십시오.(_batch_) (G7)

There are only so many draw calls, or batches of geometry, that can be submitted to GL before the application becomes CPU bound. Each draw call has an overhead that is more or less fixed. Therefore, it is very important to increase the sizes of batches whenever possible. There does not need to be a one-to-one correspondence between a draw call and a buffer—a large vertex buffer can store geometry with a similar layout for multiple models. One or more index buffers can be used to select the subset of vertices needed from the vertex buffer. A common mistake is to have too many small buffers, leading to too many draw calls and thus high CPU load. If the number of draw calls issued in any given frame goes into many hundreds or thousands, then it is time to consider combining similar geometry in fewer buffers and use appropriate offsets when defining the attribute data and creating the index buffer.
Unconnected geometry can be stitched together with degenerate triangles (alternatively, by using extension NV_primitive_restart2 when available). Degenerate triangles are triangles where two or more vertices are coincident leading to a null surface. These are trivially rejected and ignored by the GPU. The benefit from stitching together geometry with degenerate triangles, such that fewer and larger buffers are needed, tends to outweigh the minor overhead of sending degenerates triangles down the pipeline. If geometry batches are being broken up to bind different textures, then look at combining several images into fewer textures (T5).

응용 프로그램이 CPU 바운드에 걸리기 전에는, 오직 GL에 의해 제공된 수많은 드로우콜 혹은 기하의 _batch_ 돌이 있습니다. 각각의 드로우콜은 더 많거나 적게 고정된 오버헤드를 가지고 있습니다. 그러므로 가능한 만큼 _batch_ 들의 크기를 늘리는 것은 굉장히 중요합니다. 버퍼와 드로우콜 사이에 일대일 상관관계는 필요하지 않고―큰 정점 버퍼는 같은 레이아웃의 여러 모델을 저장할 수 있습니다. 하나 혹은 그 이상의 인덱스 버퍼는 정점 버퍼에서 필요한 하위의 정점을 선택하도록 사용합니다. 일반적인 실수는 수많은 작은 버퍼를 가지는 것입니다. 이는 수많은 드로우 콜, 높은 CPU 부하로 이어집니다. 만약 수많은 _drawcall_ 이 주어진 프레임에 주어지고, 그 프레임이 수백,수천만큼 이어진다면, 그때는 비슷한 기하를 몇개의 버퍼로 합치고, 어트리뷰트 데이터에 적절한 오프셋을 사용하고, 인덱스 버퍼를 만들어야 합니다. 연결되지 않은 기하는 _degenerate triangle_ 을 통해 붙여질 수 있습니다.(대안으로, NV_primitive_restart2 를 사용할 수 있습니다.) _degenerate triangle_ 은 두 혹은 더 많은 정점이 없는 표면(_null surface_)에 일치하는 삼각형 입니다.이는 보통 GPU에 의해 거부되거나 무시됩니다. 기하를 _degenerate triangle_ 을 통해 붙이는 것의 장점은, 적은 갯수의 더 큰 버퍼가 필요할 때, _degenerate triangle_ 의 부하를 능가하는 경향이 있기 때문입니다. 만약 기하 _batch_ 들이 다른 텍스쳐를 가지고 있어 부서진다면, 그때는 여러 텍스쳐럴 더 적은 텍스쳐로 합치는 것을 보세요. (T5)

## 인덱스를 위해 가장 적지만 가능한 데이터를 사용하십시오. (G8)

When the geometry uses relatively few vertices, an index buffer should specify vertices using only UNSIGNED_BYTE instead of UNSIGNED_SHORT (or an even larger integer type if the ES2 implementation supports it). Count the number of unique vertices per buffer and choose the right data type. When batching geometry for several unrelated models into fewer buffer objects (G7), then a larger data type for the indices may be required. This is not a concern compared to the larger performance benefits of batching.

기하가 상대적으로 작은 정점들을 사용할 때, 인덱스 버퍼에서 버텍스의 인덱스를 표현할 때 UNSIGNED_SHORT 대신 UNSIGNED_BYTE를 사용하세요.(혹은 ES2 구현이 제공하는 더 큰 정수 타입). 버퍼별 유일한 버텍스를 세고 적절한 데이터 타입을 선택하세요. 여러 관련되지 않은 모델를 위해 적은 버퍼 오브젝트에 있는 기하를 _batch_ 할 때, 더 큰 데이터 타입의 인덱스가 필요할 것 입니다. 이는 _batch_ 로 얻을 더 나은 성능과 비교하면 걱정할 것이 아닙니다.

## 렌더링 루프에서 새로운 버퍼를 할당하는 것을 피하십시오. (G9)

If the application frequently updates the geometry, then allocate a set of sufficiently large buffers when the application initializes. A BufferData call with a NULL data pointer will reserve the amount of memory you specify. This eliminates the time spent waiting for an allocation to complete in the rendering loop. Reusing pre-allocated buffers also helps to reduce memory fragmentation.

만약 응용 프로그램이 자주 기하를 업데이트 한다면, 그때는 충분히 큰 버퍼 세트를 응용 프로그램의 초기화시 할당하세요. null 데이터 포인터와 함께 _BufferData_ 호출은 당신이 명시한 메모리를 예약할 수 있습니다. 이는 렌더링 루프에서 할당을 완료하기 위한 시간을 제거합니다. 미리 할당한 버퍼의 재사용은 메모리 파편화를 줄이는데 도움이 됩니다.

Note: Writing to a buffer object that is being used by the GPU can introduce bubbles in the pipeline where no useful work is being done. To avoid reducing throughput when updating buffers, consider cycling between multiple buffers to minimize the possibility of updating the buffer from which content is currently being rendered.

참고: GPU에서 사용되는 버퍼 오브젝트에 쓰는 것은 파이프라인에 작업을 끝내기에 적절하지 않은 거품을 초래합니다. 버퍼에 업데이트할 때 처리량을 줄이는 것을 피하기 위해, 내용이 렌더링될 때 버퍼를 업데이트 하는 가능성을 최소화 하게 위해 여러 버퍼를 돌려가면서 쓰는 것으로 고려하세요.

## 이전에, 많이 컬링하십시오. (G10)

The GPU will not rasterize primitives when all of its vertices fall outside the viewport. It also avoids processing hidden fragments when the depth test or stencil test fails (P4). However, this does not mean that the GPU should do all the work in deciding what is visible. In the case of vertices, they need to be loaded, assembled and processed in the vertex shader, before the GPU can decide whether to cull or clip parts of the geometry. Testing a single, simple bounding volume that encloses the geometry against the current view frustum on the CPU side is a lot faster than testing hundreds of thousands of vertices on the GPU. If an application is limited by vertex processing, this is definitely the place to begin optimizing. Spheres are the most efficient volumes to test against and the volume of choice if geometry is rotational symmetrical. For some geometry, spheres tend to lead to overly conservative visibility acceptance. A rectangular cuboid (box) is only a slightly more expensive test but can be made to fit more tightly on most geometry. Hierarchical culling can often be employed to reduce the number of tests necessary on the CPU. Efficient and advanced culling algorithms have been heavily researched and published for many years. You can find a good introduction in the survey at the below locations.

 GPU는 _viewport_ 의 바깥으로 모든 정점이 떨어질 때 래스라이즈를 하지 않습니다. 이는 또한 _depth test_ 혹은 _stencil test_ 가(P4) 실패 할때 숨겨진 _fragment_ 를 피합니다. 하지만 이는 GPU가 모든 가시성을 결정하기 위한 일을 한다는 의미는 아닙니다. 정점의 경우, 로딩이 되고, 정점 쉐이더에서 조합과 처리가, GPU가 기하의 일부분을 컬링하거나 클리핑할지 말지 정하기 이전에 됩니다. 간단한 하나의 테스트, 기하를 감싸는 경계 볼륨을 현재 뷰 프러스텀을 CPU에서 비교하는 것은 GPU에서 수천개의 정점을 테스트 하는 것 보다 빠릅니다. 만약에 응용 프로그램이 정점 처리에 한계가 왔다면, 이건 최적화를 시작할 지점이 틀림없습니다. 어떤 기하의 경우, 구가 큰 보수적인 가시성 수락을 이끄는 경향이 있습니다. 직육면체는 오직 조금 더 비싼 테스트이나 대부분의 기하에 잘 맞출 수 있습니다. 위계적 컬링(hierarchical culling)은 CPU에서의 보통 테스트의 숫자를 줄이는데 종종 사용됩니다. 효율적이고 심화된 컬링 알고리즘은 몇년에 이어서 많이 연구되고 출판되었습니다. 아래의 서베이에서 좋은 소개를 볼 수 있습니다.

| Document | URL to Latest |
| ----- | ----- |
| Visibility in Computer Graphics, by Jiří Bittner and Peter Wonka | [URL](http://www.cg.tuwien.ac.at/research/publications/2003/Bittner-2003-VCG/TR-186-2-03-03Paper.pdf) |

# 쉐이더 프로그램

Writing efficient shaders is critical to achieving good performance. One should treat shaders like pieces of code that run in the inner-most loops on a CPU. There is a very high cost to littering these with conditionals or recomputing loop invariants. Before optimizing an expensive vertex shader, make sure geometry that is entirely outside the view frustum is being culled on the CPU. Before optimizing an expensive fragment shader, make sure the application is not generating an excess number of fragments with it.

효율적인 쉐이더 작성은 좋은 성능을 얻기 위해서 중요합니다. 쉐이더는 CPU에서의 루프 내부에서 실행되는 코드의 일부분으로 다루어야 합니다. 조건문과 _loop-invariant_ 를 재계산하여 쉐이더 코드를 어지르는 것은 굉장히 큰 비용이 듭니다. 값 비싼 정점 쉐이더를 최적화하기 전에, CPU에서 뷰 프러스텀 밖의 기하들을 컬링해주세요. 값 비싼 픽셀(_fragment_) 쉐이더를 최적화하기 전에, 응용 프로그램이 수많은 _fragment_ 를 생성하지 않게 하세요.

Note: When optimizing shaders, any source code that does not contribute to its output variables is optimized out by the compiler. This feature can be exploited to gain knowledge about whether the shader is part of the current bottleneck by multiplying the output variable with a null vector to reduce the workload and then measure if frame rate improves. Conversely, at the final stages of optimization one can quickly measure if there is headroom for increasing workload to offload computations to shader unit or to improve image quality by adding meaningless but expensive ALU instructions, or texture sampling, to the output variables.

참고: 쉐이더를 최적화 할때, 출력과 상관없는 모든 소스코드는 컴파일러에 의해 제외됩니다. 이 기능을 활용하여 출력 변수에 null 벡터를 곱하여 작업 부하를 줄인 다음, 프레임 속도가 향상되는지 측정하여 어떤 쉐이더가 병목의 한 부분을 차지하는지 알 수 있습니다. 반대로 최적화의 마지막 단계에서, 쉐이더 유닛에서 연산을 더 부담하거나, 값비싼 ALU 명령어들을 이미지 퀄리티를 향상시키기 위해 추가하거나, 텍스쳐 샘플링, 출력 변수들을 위한 여유분이 있는지 빠르게 측정할 수 있습니다.

## 파이프라인 윗단계로 계산을 올리십시오. (P1)

As the rendering pipeline is traversed from the CPU to the vertex processor and then to the fragment processor, the required workload tends to increase a few orders of magnitude each time. Computations constant per model, per primitive or per vertex do not belong in the fragment processor and should be moved up to the vertex processor or earlier. Per draw call computations do not belong in the vertex processor and should be moved to the CPU. For instance, if lighting is done in eye-space, the light vector should be transformed into eye-space and stored in a uniform rather than repeating this for each vertex or, even worse, per fragment. The light vector should naturally be stored pre-normalized. Usually the light vector computations are constant for the draw call, so they do not belong in any shader.

렌더링 파이프라인이 CPU에서 정점 프로세서로 그리고 _fragment_ 프로세서로 향할 수록, 필요한 처리량은 각 때마다 극단적으로 증가합니다. 모델별, 프리미티브 별 혹은 정점 별 계산 상수는 _fragment_ 처리에 속하지 않고, 반드시 정점 처리나, 그 전 단계로 이동시켜야 합니다. _drawcall_ 별 계산은 정점 처리에서 속하지 않고, CPU로 옮겨야 합니다. 예를 들어, _lighting_ 이 _eye-space_ 에서 끝난다면, 광원 벡터는 반드시 _eye-space_ 로 변환되어야 하고, 각각의 정점별로, 더 안좋은 경우 _fragment_ 별로 반복하는 것 보다 _uniform buffer_ 에 저장해야 합니다. 광원 벡터는 자연스럽게 미리 정규화되어 저장되어야 합니다. 보통 광원 벡터 계산은 _drawcall_ 마다 상수이므로 어떤 쉐이더에도 포함되지 않습니다.

## 크거나 일반화된 쉐이더를 작성하지 마십시오. (P2)

It is critical to resist the temptation to write shader programs that take different code paths depending on whether one or more constant variable have a particular value. Uniforms are intended as constants for one (or hopefully many) primitives—they are not substitutes for calling UseProgram. Shaders should be minimal and specialized to the task they perform. It is much better to have many small shaders that run fast than a few large shaders that all run slow. Code re-use (when source shaders are supported) should be handled at the application level using the ShaderSource function. If the advice here of not writing generalized shaders goes against the conflicting goal of minimizing shader and state changes, smaller and more specialized shaders are generally preferred. Additionally, be careful with writing shader functions intended for concatenation into the final shader source code - shared functions tend to be overly generic and make it harder to exploit possible shortcuts.

상수가 특정 값을 가지냐에 따라 코드 분기를 달리하는 쉐이더를 작성하고 싶은 유혹에 저항하는 것은 굉장히 중요합니다. _uniform buffer_ 는 하나(다다익선) 프리미티브에 대한 상수로 사용되며, `UseProgram` 을 대체하진 않습니다. 쉐이더는 반드시 목적에 맞게 특화되고 최소화 되어야 합니다. 느리게 실행되는 큰 쉐이더 몇개 보다, 빠르게 실행되는 수많은 작은 쉐이더가 더 낫습니다. 코드 재사용은 (소스 쉐이더 프로그램이 지원될 때) `ShaderSource` 함수를 통하여  응용 프로그램 레벨에서 제어되어야 합니다. 여기서 `일반화된 쉐이더를 작성하지 마라` 라는 조언이 쉐이더 크기와 상태 변화를 최소화 하는 목표와 상충된다면, 일반적으론 더 작고 특화된 쉐이더 작성을 선택합니다. 또한, 마지막 쉐이더 소스 코드에 덧붙일 쉐이더 함수를 사용할 때 주의해야 합니다. 쉐이더 함수는 심하게 일반화 되고, 가능한 _shortcut_ 을 이용하는 것을 어렵게 만듦니다.

## 응용프로그램의 특정한 지식을 활용하십시오. (P3)

Application specific knowledge can be used to simplify or avoid computations. Math shortcuts should be pursued everywhere because there are optimizations that the shader compiler or the GPU cannot make. For instance, rendering screen-aligned primitives is common for 2D user interface and post-processing effects. In this case, the entire modelview transformation is avoided by defining the vertices in NDC (normalized device coordinates). A full-screen quad has its vertex coordinates in the [-1.0,1.0] range so these can be passed directly from the vertex attribute to gl_Position. The types of matrix transformations applied in the application when creating the modelview matrix should be tracked and exploited when possible. For instance, an orthonormal matrix (e.g. no non-uniform scaling) leads to an opportunity to avoid computations when transforming normals with the inverse-transpose sub-matrix.

특정한 응용 프로그램만의 케이스는 계산을 피하거나 단순화할 수 있습니다. 쉐이더 컴파일러나 GPU 가 만들 수 없는 수학적 계산 최적화가 있기 때문에 모든 곳에서 신경써야합니다. 예를 들면, _screen-aligned primitive_ 는 (화면에 정렬된 도형) 2D UI 나 _post-processing_ 효과에 자주 쓰입니다. 이 경우, 모든 모델-뷰 변환은 정점을 미리 NDC(normalized device coordinates) 로 정의하여 피할 수 있습니다. 전체 화면 _quad_ 는 (사각 폴리곤) \[-1.0, 1.0\] 범위에서 정점 위치를 가지기때문에 _vertex attribute_ 에서 직접 _gl\_Position_ 으로 통과할 수 있습니다. 이 행렬 변환의 타입들은 모델-뷰 행렬을 만드는 것이 가능한한 추적되거나 활용될 때 응용 프로그램에서 적용 됩니다. 예를 들어 _orthonormal_ (정규직교) 행렬은 _inverse-transpose_ 부 행렬과 함께 법선 벡터를 변환시킬 때, 계산을 피할 수 있는 기회를 이끌어 냅니다.

## 깊이, 스텐실 컬링을 최적화하십시오. (P4)

The GPU can quickly reject fragments based on depth or stencil testing before the fragment shader is executed. The depth complexity of a scene is the number of times each fragment gets written. Depth complexity can be measured by incrementing values in a stencil buffer. A high depth complexity in a 3D scene can be a result of rendering opaque objects in a non-optimal order. The worst case is rendering back-to-front (aka painter's algorithm) because it leads to a large number of fragments being overdrawn. An application with high depth complexity should ensure that opaque objects are rendered sorted front-to-back order with depth testing enabled. Straightforward rendering of 2D user interfaces also leads to a high depth complexity that can often be decreased with the same technique but also by using the stencil buffer to mask fragments. Applications that are heavily fragment limited can be sped up significantly with clever use of these techniques—sometimes up to a factor of 10 or more.

GPU 는 _fragment_ 쉐이더가 실행되기 전, 깊이 혹은 스텐실 테스팅을 기반으로 _fragment_ 쉐이더를 빠르게 제외할 수 있습니다. 환경의 _depth complexity_ (깊이 복잡도)는 각각의 _Fragment_ 에 쓰는 횟수입니다. _depth complexity_ 는 스텐실 버퍼를 1씩 증가시켜가며 측정 가능합니다. 3D 환경에서 높은 _depth complexity_  은 최적의 순서가 아닌 불투명 오브젝트 렌더링의 결과로 나타날 수 있습니다. 가장 나쁜 경우는 _back-to-front_ 렌더링 입니다. (다른 말로는, 화가의 알고리즘) 왜냐면, 이는 많은 _fragment_ 들을 덮어씌우기 때문입니다. 높은 _depth complexity_ 를 지닌 응용 프로그램은 반드시 _depth test_ (깊이 검사) 와 함께 정렬된 _front-to-back_ 순서로 불투명 오브젝트의 렌더링을 보장해야 합니다. 2D UI 의 단순한 렌더링 또한 같은 테크닉 뿐만 아니라 _fragment_ 마스킹을 위해 스텐실 버퍼를 이용하여 높은 _depth complexity_ 를 줄일 수 있습니다. 심하게 _fragment_ 비용이 제한된 응용 프로그램은 이러한 똑똑한 방법의 사용으로 속도를 10배 혹은 그 이상 높일 수 있습니다.

If vertex processing is not a bottleneck, it is worthwhile to run experiments that prime the depth buffer in a first pass. Disable all color writes with ColorMask on the first pass. The fragments in the depth buffer can then serve as occluders in a second pass when color writes are enabled and the expensive fragment shaders are executed. Disable depth writes with DepthMask in the second pass since there is no point in writing it twice.

만약 정점 처리가 병목이 아니라면, 첫번째 _pass_ 에서 깊이 버퍼를 쓰는 실험을 실행하는 것이 의미 있습니다. 첫 _pass_ 에서 _ColorMask_ 와 함께 모든 색 쓰기를 비활성화합니다. 깊이 버퍼의 _fragment_ 는 색 쓰기가 가능하고, 비싼 _fragment_ 쉐이더가 실행될 때, 두번째 _pass_ 에서 _occluders_ (가리는 오브젝트) 를 대신할 수 있습니다. 두번이나 쓸 필요는 없기 때문에, 두번째 _pass_ 에서는 _DepthMask_ 와 함께 깊이 쓰기를 비활성화 합니다.

## 픽셀 처리 조각(_fragment_)를 버리거나, 깊이를 수정하지 마십시오. 완전히 필요한게 아니라면, (P5)

Some operations prevent the hardware from enabling its automatic optimization that rejects fragments early in the pipeline (early-Z). In particular, the discard operation that discards fragments based on some criteria will disable early-Z on some platforms. It is critical to limit the use of discarding as much as possible (e.g., alpha testing)—unless depth writing can be disabled. Another example is found in the GL_NV_fragdepth extension available on some platforms, where the depth value can be written from the fragment shader. This operation also forces the GPU to opt out of Early-Z reject in order to ensure correct rendering.

몇몇 명령은 파이프라인에서 일찍 _fragment_ 연산을 제외하는(ealry-Z) 자동 최적화를 하드웨어가 하지 못하도록 합니다. 특히, 특정 기준에 따라서 _fragment_ 를 버리는 _discard_ 명령은 몇몇 플랫폼에서 _early-Z_ 를 비활성화 시킵니다. 깊이 쓰기를 비활성화 할 수 없는 경우, _discard_ 연산을 최대한 제한하는 것은 중요합니다. 다른 예시를 깊이 값을 _fragment_ 쉐이더에서 쓰기 가능하고, 특정 플랫폼에서 가능한 _GL\_NV\_fragdepth_ 확장에서 찾을 수 있습니다. 이 명령은 또한 정확한 렌더링을 보장하기 위해 _early-Z_ 거부를 강제로 비활성화 시킵니다.

## 조건문을 최대한 피하십시오. (P6)

Fragments are processed in chunks and both branches of a conditional may need to be evaluated before the result of the false branch can be discarded by the GPU. Be careful with assuming that conditionals skip computations and reduce the workload. This warning is particularly relevant to fragment shaders. Benchmarking shaders can determine if conditionals in the vertex or fragment shaders actually end up decreasing the workload. Some conditional code can be rewritten in terms of algebra and/or built-in functions. For instance, the dot product between a normal and a light vector may be negative in which case the result is not needed in a lighting equation. Instead of:

GPU에 의해 _false branch_ 의 (실행되지 않을 코드 흐름) 결과를 제외하기 전, _fragment_ 들은 한꺼번에 처리되고, 각각의 분기내의 코드 흐름을 아마 평가할 필요가 있습니다. 조건문이 계산을 건너 뛴거나, 워크로드를 줄인다는 가정을 조심하십시오. 이 경고는 특히 _fragment_ 쉐이더와 연관되어 있습니다. 쉐이더 벤치마킹은 정점 혹은 _fragment_ 쉐이더에서의 조건이 실질적으로 끝내어 워크로드를 줄이는지에 따라 결정됩니다. 몇몇 조건문 코드는 대수 and/or 내장 함수로 다시 쓰일 수 있습니다. 예를 들어 법선 벡터와 빛의 방향 벡터를 내적의 결과가 음수 일 수 있으며, 이 경우 _light equation_ 내에서 결과가 필요하지 않습니다. 

대신에:

```
if (nDotL > 0.0) ...
```
the value can be clamped with:

이 값은 _clamp_ 로 대체할 수 있습니다:

```
clamp(nDotL, 0.0, 1.0)
```

and unconditionally used in the result (the negative value results in a zero-product). Clamp may be faster than max and/or min for the 0.0 and 1.0 cases, but as always benchmarking will have the final say in the matter. Another reason to make an effort of avoiding conditionals in fragment shaders is that mipmapped textures return undefined results when executed in a block statement that is conditional on run-time values. Although the GLSL functions `texture*Lod` can be used to bias or specify the mipmap LOD, it is expensive to manually derive the mipmap LOD. In addition, these LOD biasing samplers may not run as fast as the non-LOD samplers.

그리고 조건문 없이 결과에 사용될 수 있습니다.(음수가 0 곱하기로 표현되는 경우) _clamp_ 연산은 아마도 _max_ ,_min_ 을 0과 1사이에서 실행하는 것보다 빠를 수 있습니다. _fragment_ 쉐이더에서 조건문을 피하기 위한 노력의 다른 이유는 밉맵을 가진 텍스쳐는 런타임에 달라지는 조건에 따른 _block statement_ (조건에 블록 형태로 존재하는 문장: 여러 코드 줄) 에서 실행 될 때, 정의되지 않은 결과를 반환하기 때문입니다. GLSL 함수 `texture*Lod`를 사용하여 밉맵 LOD 를 _bias_ (오프셋 처리) 하거나, 지정할 수 있지만, 직접 밉맵 LOD 를 파생시키는 것은 비쌉니다. 또한 이러한 _LOD-biasing_ 샘플러는 _LOD-biasing_ 이 아닌 샘플러보다 빠르지 않을 것입니다.

## 적절한 정확도 한정자를 사용하십시오. (P7)

Recall that the default precision in vertex shaders is highp, and that fragment shaders have no default precision until explicitly set. Precision qualifiers can be valuable hints to the compiler to reduce register pressure and may improve performance for several reasons. Low precision may run twice as fast in hardware as high precision. These optimizations can be approached by initially using highp and gradually reducing the precision to lowp until rendering artifacts appear; if it looks good enough, then it is good enough. As a rule of thumb, vertex position, exponential and trigonometry functions need highp. Texture coordinates may need anything from lowp to highp depending on texture width and height. Many application-defined uniform variables, interpolated colors, normals and texture samples can usually be represented using lowp. However, floating-point texture samplers need more than low precision - this is one of several reasons to minimize the use of floating-point textures (T6).

정점 쉐이더에서 기본 부동소수점 정확도는 _highp_ 이지만, _fragment_ 쉐이더는 기본 부동소수점 정확도는 명시저긍로 정해주어야 합니다. 정확도 한정자는 컴파일러에게 레지스터 압박을 줄이거나, 여러 이유로 성능을 향상시킬 수 있는 가치있는 힌트가 될 수 있습니다. 낮은 정확도는 높은 정확도 보다 하드웨어에서 두배로 빠를 것 입니다. 이런 최적화는 처음에는 _highp_ 를 사용하고, 점진적으로 정확도를 _lowp_  로 렌더링 _artifact_ 가 생길 때까지 줄여나가며 접근할 수 있습니다. 그 _artifact_ 에도 충분하다면, 그것은 충분한 것입니다. 경험적으로, 정점의 위치, 지수 및 삼각 함수는 _highp_ 가 필요합니다. 텍스쳐 좌표는 텍스쳐의 크기에 따라서, _lowp_ 부터 _highp_ 까지 다르게 필요합니다. 많은 응용 프로그램에서 정의된 _uniform variables_, _interpolated colors_, _normals_, _texture samples_ 는 보통 _lowp_ 를 사용하여 표현될 수 있습니다. 하지만 부동 소수점 텍스쳐 샘플러는 _lowp_ 보다 나은 정확도가 필요합니다. 이는 여러 부동 소수점 텍스쳐의 사용을 최소화해야되는 여러 이유 중에 하나입니다. (T6)

## 빌트인 함수와 변수를 사용하십시오. (P8)

The built-ins have a higher chance of compiling optimally and may even be implemented in hardware. For instance, do not write shader code to determine the forward primitive face or compute the reflection vector in terms of dot-products and algebra; instead, use the built-in variable gl_FrontFacing or the built-in function reflect, respectively.

빌트인 함수 및 변수는 최적의 컴파일의 중요한 기회 가지고, 하드웨어에서 구현된 것일 수도 있습니다. 예를 들어, _primitive_ 정면 혹은 반사 벡터를 내적 그리고 대수로 결정하는 쉐이더 코드를 작성하는 것 대신, _gl_FrontFacing_ 혹은 _reflect_ 빌트인 함수를 사용하세요.

## 복잡한 함수는 텍스쳐에 인코딩하는 것을 고려하십시오. (P9)

Shaders normally contain both arithmetic (ALU) and texture operations. A batch of ALU operations may hide the latency of fetching samples from texture because they occur in parallel. If a shader is the primary bottleneck, and when the ALU operations significantly outnumber the texture operations, it is worthwhile to investigate if some of these operations can be encoded in textures. Sub-expressions in shaders can sometimes be stored in LUTs (look-up-tables). LUTs can sometimes be implemented in textures with sufficient precision and accessed as 1D or 2D textures with NEAREST filtering.

보통 쉐이더는 계산 유닛과(ALU) 텍스쳐 유닛을 둘다 가집니다. ALU 명령의 _batch_ 는 (한꺼번에 처리) 병렬적으로 실행되기 때문에 텍스쳐에서 샘플을 가져오는 지연 시간을 감춥니다. (hide of latency) 만약 쉐이더가 주요한 병목이라면, ALU 명령이 텍스쳐 명령보다 훨씬 많을 때, 특정 연산을 텍스쳐에 인코딩 하는 것을 조사하는 것은 의미있는 일입니다. 쉐이더의 일부(sub-expression)은 가끔 LUT들에 (look-up-table)저장될 수 있습니다. LUT는 가끔 충분한 정확도 그리고 NEAREST 필터링과 함께 1D 혹은 2D 텍스쳐로 구현될 수 있습니다.

Note: The old trick of using cubemaps to normalize vectors is most likely a performance loss on discrete GPUs. If you pursue this idea, then make sure to benchmark to determine if you have improved or worsened the performance!

참고: 벡터 정규화를 위해 큐브맵을 사용하는 오래된 트릭은 discrete GPU 에서 성능 손실을 가져올 가능성이 높습니다. 만약 당신이 이를 주장한다면, 성능을 향상시킬지, 나쁘게 하는 것인지 결정하기 위해 벤치마크를 하세요.


## 간접 텍스쳐링의 사용을 제한하십시오. (P10)

Indirect texturing can sometimes be useful, but when the result of a texture operation depends on another texture operation, the latency of texture sampling is difficult to hide. It also tends to lead to scattered reads that minimize the benefit of the texture cache. Indirect texturing can sometimes be reduced, or avoided, at the expense of memory. Whether that trade-off makes sense should of course be analyzed and benchmarked.

간접 텍스쳐링은 가끔 유용하지만, 텍스쳐 연산의 결과가 다른 텍스쳐 연산에 의존한다면, 텍스쳐 샘플링의 지연 시간을 감추기 어렵습니다. 또한 텍스쳐 캐시의 장점을 최소화 하는 파편화된 읽기를 이끄는 경향이 있습니다. 간접 텍스쳐링은 가끔은 메모리 비용 면에서 줄이거나, 피할 수 있습니다. 이 트레이드 오프가 타당한지 분석하고, 벤치마크 되어야 합니다.

## 수치 계산 최적화를 모호하게 GLSL 구문을 작성하지 마십시오. (P11)

The GLSL shading language conveniently overloads arithmetic operators for vectors and matrices. Care must be taken to not miss optimization opportunities due to this syntax simplification. For instance, a rotation matrix can, but should not, be defined as homogenous 4x4 matrix just because the other operand is a vec4 vector. A generalized rotation matrix should be described only as a 3 x 3 matrix and applied to vec3 vectors. And a rotation around basic vectors can be done even more efficiently than mat3 * vec3 by directly accessing the relevant vector and matrix components with cos and sin. Take advantage of any specific application knowledge to reduce the number of needed scalar instructions.

GLSL 쉐이딩 언어는 벡터와 행렬을 위해 산수 계산을 편리하게 오버로드합니다. 이 syntax 단순화 때문에 최적화 기회를 날리지 않기 위해 신경써야 합니다. 예를 들어, 회전 행렬은 가능하지만, 불가능 할 수도 있는데, 동차 좌표계의 4x4 행렬으로 다른 인자가 vec4 이기 때문에 정의될 수 있습니다. 일반화된 회전 행렬은 오직 3x3 행렬로 묘사되고 vec3 벡터에 적용되어야 합니다. 그리고 기본 벡터의 회전은 직접 관련된 벡터 및 행렬의 요소에 cos와 sin과 함께 직접 접근하여 mat3 * vec3 보다 더 효율적으로 끝낼 수 있습니다. 필요한 스칼라 명령의 수를 줄이기 위해 응용 프로그램의 특정 지식을 이용하세요.

## 정규화는 필요할 때만 하십시오. (P12)

Normalization of vectors is required to efficiently calculate the angle between them, or perhaps more typically, the diffuse component in many commonly used lighting equations. In theory, normalization is required for geometry normals after having transformed them with the normal matrix. In practice, it may not matter depending on the composition of the matrix. It is a common mistake to normalizing vectors where it is not necessary, or at least not visually discernible. As a result, the application may run slower for no good reason. For instance, consider the case of interpolating vertex normals in order to compute lighting per fragment (i.e., Phong shading). If the normal matrix only rotates, there is little reason to normalize the normal vectors before interpolating.

벡터의 정규화는 두 벡터의 각이나, 아마도 더 일반적으로, 가장 많이 사용되는 _lighting equation_ 의 _diffuse_ 컴포넌트에서 효율적으로 계산하기 위해 필요합니다. 이론상으로, 법선 행렬과 함께 변환된 이후에 지오메트리 법선 벡터를 위해 정규화가 필요합니다. 실전에서는 행렬의 합성에서 딱히 별일이 없습니다. 이는 필요하지 않거나, 적어도 시각적으로 보고 알 수 있지 않아도 정규화를 하는 일반적인 실수입니다. 결과적으론, 별 이유 없이 응용 프로그램은 더 느려집니다. 예를 들어 정점 법선 벡터를 _fragment_ 별 라이팅 계산을 위해 보간(정점 단계 -> 픽셀 단계에서, 래스터라이저를 거치며 일어나는) 하는 경우를 생각해 봅시다.(즉, 퐁 쉐이딩) 만약에 법선 행렬이 오직 회전만 한다면, 보간 전에 법선 벡터를 정규화할 조그만 이유가 있습니다. 

Note: Barycentric interpolation will not preserve the unit length of these vectors. So normals that are interpolated in varying variables do must be normalized to ensure the dot-product with the light vector obeys the cosine emission law.

참고: 무게 중심 보간은 정규화를 보장하지 못합니다. 그래서 _varying variable_ 에서 (래스터라이저를 이후에) 보간된 법선 벡터는 _lambert cosine law_ 를 따르는 광원 벡터와의 내적을 보장하기 위해 정규화를 해야 합니다.

# 텍스쳐

Textures consume the largest amount of the available memory and bandwidth in many applications. One of the best places to look for improvements when short on memory or bandwidth to optimize texture size, format and usage. Careless use of texturing can degrade the frame rate and can even result in inferior image quality.

텍스쳐는 많은 응용 프로그램에서 가장 많은 가용 메모리와 대역폭을 소모합니다. 메모리와 대역폭이 부족할 때, 텍스쳐 크기, 포맷 및 사용을 최적화하여 성능 향상을 꾀할 수 있는 가장 좋은 곳 중 하나입니다. 무분별한 텍스쳐의 사용은 프레임을 감소시키고, 심지어 이미지 품질이 저하될 수 도 있습니다.

## 텍스쳐 압축은 가능한 사용하십시오. (T1)

Texture compression brings several benefits that result from textures taking up less memory: compressed textures use less of the available memory bandwidth, reduces download time, and increases the efficiency of the texture cache. The texture_compression_s3tc and texture_compression_latc extensions both provide block-based lossy texture compression that can be decompressed efficiently in hardware. The S3TC extension gives 8:1 or 4:1 compression ratio and is suitable for color images with 3 or 4 channels (with or without alpha) with relatively low-frequency data. Photographs and other images that compress satisfactory with JPEG are great candidates for S3TC. Images with hard and crisp edges are less good candidates for S3TC and may appear slightly blurred and noisy. The LATC extension yields a 2:1 compression ratio, but improves on the quality and can be useful for high resolution normal maps. The third coordinate is derived in the fragment shader—be sure to benchmark if the application can afford this trade-off between memory and computations! Unlike S3TC, the channels in LATC are compressed separately, and quantization is less hard. Using texture compression does not always result in lower perceived image quality, and with these extensions one can experiment with increasing the texture resolution for the same memory. There are off-line tools to compress textures (even if a GL extension supports compressing them on-the-fly). Search the NVIDIA developer website for "Texture Tools".

텍스쳐 압축은 적은 메모리를 사용하는 텍스쳐로 부터 여러 이익을 가져다 줍니다: 압축된 텍스쳐는 적은 메모리 대역폭, 적은 다운로드 시간 및 텍스쳐 캐시 효율을 증대시켜 줍니다. `texture_compression_s3tc` 와 `texture_compression_latc` 확장은 둘 다 하드웨어에서 효율적으로 해제 가능한 블록 기반의 손실 텍스쳐 압축을 제공합니다. `S3TC` 확장은 8:1 혹은 4:1 압축 비율을 가지며, 비교적 낮은 주파수의 데이터를 가진 3 혹은 4 채널의 색 이미지에 적합합니다. JPEG를 사용하여 만족할 정도로 압축된 사진이나 다른 이미지는 `S3TC` 의 좋은 후보입니다. 튀고 뾰족한 선을 가진 이미지는 `S3TC` 의 덜 좋은 후보이고, 아마도 희미하거나, 노이즈가 끼어있을 수 있습니다. `LATC` 확장은 2:1 압축 비율을 가지지만, 이는 질을 향상시키고, 높은 해상도의 노말맵에 유용합니다. 세번째 좌표는 _fragment_ 쉐이더에서 유도됩니다-반드시 메모리와 계산 사이에서 응용 프로그램이 수용 가능한지 벤치마크하세요! `S3TC` 와 다르게 `LATC`에서는 채널을 분리하여 압축하고, 양자화는 비교적 덜 강하게 됩니다. 텍스쳐 압축을 쓰는 것은 무조건 낮은 퀄리티의 이미지를 만드는게 아니기 때문에 같은 메모리를 사용할 때 해상도를 증가시키는 실험을 앞선 확장과 함께 할 수 있습니다. 오프라인 텍스쳐 압축 툴도 있습니다.(심지어 GL 확장은 바로 압축 또한 지원합니다.) 

## 적절하게 밉맵을 사용하십시오. (T2)

Mipmaps should be used when there is not an obvious one-to-one mapping between texels and framebuffer pixels. If texture minification occurs in a scene and there are no mipmaps to access, texture cache utilization will be poor due to the sparse sampling. If texture minification occurs more often than not, then the texture size may be too large to begin with. Coloring the mipmap levels differently can provide a visual clue to the amount of minification that is occurring. When reducing the texture size, it may also be worthwhile to perform experiments to see if some degree of magnification is visually acceptable and if it improves frame rate.

밉맵은 _texel_ 과 프레임버퍼 픽셀간의 1대1 매핑을 중요하지 않은 경우엔 사용되어야 합니다. 만약 환경에서 텍스쳐 축소가 일어나고 밉맵 접근이 불가능하다면, 텍스쳐 캐시 활용은 드문 드문 샘플링하는 것 때문에 나쁜 성능을 가질 것입니다. 만약 텍스쳐 축소가 자주 일어난다면, 텍스쳐 크기는 시작부터 너무 큰 크기를 가질 것입니다. 밉맵 레벨별로 다르게 색칠하는 것은 축소가 얼마나 일어나는지에 대한 시각적 증거를 제공합니다. 또한 텍스쳐 크기를 줄일 때, 어느 정도의 확대가 시각적으로 괜찮은지, 얼마나 _frame rate_ 를 올려줄 지 보기 위해 실험을 하는 것 또한 가치있을 것입니다.

Although the GenerateMipmap function is convenient, it should not be the only option for generating a mipmap chain. This function emphasizes execution speed over image quality by using a simple box filter. Generating mipmaps off-line using more advanced filters (e.g. Lanczos/Sinc) will often yield improved image quality at no extra cost. However, GenerateMipmap may be preferable when generating textures dynamically due to speed. One of the only situations where you do not want to use mipmaps is if there is always a one-to-one mapping between texels and pixels. This is sometimes the case in 3D, but more often the case for 2D user interfaces. Recall that a mipmapped texture takes up 33% more storage than un-mipmapped, but they can provide much better performance and even better image quality through reduced aliasing.

`GenerateMipmap` 함수가 간편함에도 불구하고, 이는 밉맵 체인을 생성하는 단 하나만의 옵션은 아닙니다. 이 함수는 간단한 _box filter_ 를 사용하여 이미지 퀄리티와 상관 없이 실행 속도를 강조합니다. 오프라인에서 더 나은 필터를 사용하는 밉맵 생성은 보통 추가적인 비용 없이 이미지 퀄리티를 올릴 수 있습니다. 하지만 _GenerateMipmap_ 은 동적으로 텍스쳐를 만들어 낼 때, 아마 속도 때문에 선호됩니다. 당신이 밉맵을 사용하지 않을 상황은 텍셀과 픽셀이 언제나 일대일 매핑되어야 할 때 입니다. 3D 환경에서 근근히 있는 케이스이지만, 2D UI에서 더 자주 있는 일입니다. 정리하자면, 밉맵이 있는 텍스쳐가 33%의 메모리를 더 차지하지만, 더 나은 성능과 심지어 줄여진 _aliasing_ 을 통해 더 나은 이미지 퀄리티를 제공할 수 있습니다.

## 가능한 최소한의 텍스쳐 사이즈를 사용하십시오. (T3)

Always use the smallest possible texture size for any content that gives acceptable image quality. The appropriate size of a texture should be determined by the size of the framebuffer and the way the textured geometry is projected onto it. But even when you have a one-to-one mapping between texels and framebuffer pixels, there may be cases where a smaller size can be used. For instance, when blending a texture on the existing content in the entire framebuffer, the texture does not necessarily have to be the same width and height as the framebuffer. It may be the case that a significantly smaller texture that is magnified will produce results that are good enough. The bandwidth that is saved from using a smaller and more appropriately sized texture can instead be spent where it actually contributes to better image quality or performance.

무엇이든 간에 수용 가능한 이미지 퀄리티를 제공하는 선에서 항상 최소한의 가능한 텍스쳐 사이즈를 사용해야 합니다. 텍스쳐의 적절한 크기는 프레임 버퍼의 크기와 텍스쳐가 입힌 기하가 어떻게 투영되는지에 따라 결정되어야 합니다. 하지만 심지어 _texel_ 과 프레임 버퍼의 픽셀의 일대일 매핑을 원할 때도 사용 가능한 가장 작은 크기를 고르게 될 것입니다. 예를 들어 있던 전체 프레임 버퍼와 텍스쳐를 블렌딩 할 때, 텍스쳐는 프레임 버퍼와 같은 가로,세로 크기를 가져야 합니다. 아마도 확대된 상당히 작은 텍스쳐가 충분히 괜찮은 결과를 만들 수 있습니다. 더 작고 적절한 크기의 텍스쳐를 사용하여 얻은 대역폭은 실질적으로 더 나은 이미지 퀄리티와 성능을 위해 대신 사용될 수 있습니다.

## 가능한 최소한의 텍스쳐 포맷과 데이터 타입을 사용하십시오. (T4)

If hardware accelerated texture compression cannot be used for some textures, then consider using a texture format with fewer components and/or fewer bits per component. Textures for user interface elements sometimes have hard edges or color gradients that result in inferior image quality when compressed. The S3TC algorithm make assumptions that changes are smooth and colors values can be quantized. If these assumptions do not fit a particular image, but the number of unique colors is still low, then experiment with storing these in a packed texture format using 16 bit/texel (e.g. UNSIGNED_SHORT_5_6_5). Although the colors are remapped with less accuracy it may not be noticeable in the final application. Grayscale images should be stored as LUMINANCE and tinted images can sometimes be stored the same way with the added cost of a dot product with the tint color. If normal maps do not compress satisfactory with the LATC format, then it may be possible to store two of the normals coordinates in uncompressed LUMINANCE_ALPHA and derive the third in a shader assuming the direction (sign) of the normal is implicit (as is the case of a heightmap terrain).

특정 텍스쳐에서 하드웨어 가속 압축을 사용할 수 없을 때, 컴포넌트나 컴포넌트 별 비트 수를 줄이는 것을 고려해 보세요. UI 요소를 위한 텍스쳐는 가끔 압축을 사용할 때 낮은 이미지 퀄리티를 보여주는 하드 엣지나 컬러 그래디언트를 가질때가 가끔 있습니다. `S3TC` 알고리즘은 부드럽고 양자회 될 수 있는 색을 변화시킨다고 가정합니다. 만약 이 가정이 특정 이미지에 맞지 않고 유일한 색의 갯수는 여전히 적다면, _texel_ 별 16비트를 사용하는 패킹된 텍스쳐 포맷을 사용하여 이를 저장하는 실험을 해보세요.(예를들어 `UNSIGNED_SHORT_5_6_5`) 색이 조금 부정확해지더라도, 마지막 결과에서는 눈에 띄지 않을 것입니다. 그레이스케일 이미지는 `LUMINANCE` 로 저장되어야 하고, _tinted_ 이미지의 경우 가끔 _tint color_ 와 내적하는 비용과 함께 같은 방법으로 저장될 수 있습니다. 노말 맵을 `LATC` 포맷으로 만족할만큼 압축하지 못한다면, 압축되지 않은 `LUMINANCE_ALPHA` 에 두 좌표값을 저장하고, 세번째 좌표값은 쉐이더에서 암시적인 방향을 설정하여 계산할 수 있습니다.(heightmap 지형의 경우)

Note: When optimizing uncompressed textures, the exception case that 24-bit (RGB) textures are not necessarily faster to download or smaller in memory than 32-bit (RGBA) on most GPUs. In this case, it may be possible to use the last component for something useful. For instance, if there already is an 8-bit greyscale texture that is needed at the same time as an opaque color texture, that single component texture can be stored in the unused alpha component of a 32-bit (RGBA). The component could define a specular/reflectance map that describe where and to what degree light is reflected. This is useful for terrain satellite imagery or land cover textures with water/snow/ice areas or for car textures with their metal and glass surfaces or for textures for buildings with glass windows.

참고: 압축되지 않은 텍스쳐를 최적화할 때, 24-bit(RGB) 텍스쳐가 빠르게 다운로드 하지 않아도 되거나, 32-bit(RGBA) 보다 메모리가 작을 때는 대부분의 GPU에선 예외입니다. 이 경우, 마지막 컴포넌트를 유용하게 사용할 수 있을 것입니다. 예를들어 이미 8-bit 그레이스케일 텍스쳐가 불투명 색 텍스쳐와 함께 사용될 때, 32-bit(RGBA)의 미사용 알파 컴포넌트를 사용할 수 있습니다. 이 컴포넌트는 어디에, 어느 각도에 빛이 반사되는지 사용하기 위한 _specular_ 혹은 _refelectance_ 맵으로 정의할 수 있습니다. 이는 지형 위성 이미지 생성, 땅의 물/눈/얼음 영역, 차의 철과 유리 표면의 구분, 빌딩의 유리창의 구분에 유용하게 사용될 수 있습니다.

## 각각의 텍스쳐 오브젝트에 여러 이미지를 저장하십시오. (T5)

There is no requirement that there is a one-to-one mapping between an image and a texture object. Textures objects can contain multiple distinct images. These are sometimes referred to as a "texture atlas" or a "texture page". The geometry defines texture coordinates that only reference a subset of the texture. Texture atlases are useful for minimizing state changes and enables larger batches when rendering. For example, residential houses and office buildings and factories might all use distinct texture images. But the geometry and vertex layout for each is most likely identical so these could share the same buffer object. If the distinct images are stored in a texture atlas instead of as separate textures, then these different kinds of buildings can all be rendered more efficiently in the same draw call (G7). The texture object could be a 2D texture, a cubemap texture or an array texture.

이미지와 텍스쳐 오브젝트 간의 반드시 일대일 매핑이 되어야 한다는 요구 사항은 없습니다. 텍스쳐 오브젝트는 여러 구분가능한 이미지를 가질 수 있습니다. 이는 텍스쳐 아틀라스 혹은 텍스쳐 페이지로 종종 불립니다. 기하는 텍스쳐 내의 참조만을 위한 텍스쳐 좌푤르 저장합니다. 텍스쳐 아틀라스는 상태 변화를 최소화 시키고, 한꺼번에 더 많은 렌더링을 가능하게 합니다.(larger batch) 예를들어 단독 주택, 사무실 빌딩 그리고 공장은 서로 다른 텍스쳐 이미지를 사용합니다. 하지만 각각의 기하와 정점 레이아웃은 거의 비슷하기 때문에 같은 버퍼 오브젝트를 공유할 수 있습니다. 만약 다른 이미지가 분리된 텍스쳐 대신 텍스쳐 아틀라스에 저장된다면, 다른 종류의 빌딩이 같은 드로우콜에서 효율적으로 전부 렌더링될 수 있습니다.(G7) 텍스쳐 오브젝트는 2D 텍스쳐, 큐브맵 텍스쳐 혹은 배열 텍스쳐가(array texture) 될 수 있습니다. 

Note: Note that if mipmapping is enabled, the sub-textures in an atlas must have a border wide enough to ensure that smaller mipmaps are not generated using texels from neighboring images. And if texture tiling (REPEAT or MIRRORED_REPEAT) is needed for a sub-image then it may be better to store it outside the texture atlas.

참고: 밉맵이 활성화 되어 있다면, 각 아틀라스별 실질적인 텍스쳐는 근처 텍셀을 통해 밉맵이 만들어지지 않도록 반드시 적절히 넓은 경계를 가져야 합니다. 만약 텍스쳐 타일링(REPAT, MIRRORED_REPEAT) 이 실질적인 텍스쳐에서 필요하다면 텍스쳐 아틀라스의 바깥에 저장하는 것이 나을 것 입니다.

Emulating either wrapping mode in a shader by manipulating texture coordinates is possible, but not free. A cubemap texture can sometimes be useful since wrapping and filtering apply per face, but the texture coordinates used must be remapped to a vec3 which may be inconvenient. If all the sub-images have the same or similar size, format and type (e.g. image icons), the images are a good candidate for the array texture extension if supported. Array textures may be more appropriate here than a 2D texture atlas where mipmapping and wrapping restrictions have to be taken into consideration.

텍스쳐 좌표를 사용하여 쉐이더 내에서 _wrapping mode_ 를 조작하는 것은 가능하나, 공짜는 아닙니다. 큐브맵 텍스쳐는 _wrapping_ 및 _filtering_ 을 표면별로 해주기 때문에 종종 유용하지만, 텍스쳐 좌표가 반드시 vec3 으로 만들어져야해서 불편합니다. 만약 모든 실질적인 텍스쳐가 같거나 더 작은 크기, 포맷 그리고 타입을 가진다면, 이미지는 만약 제공된다면 _array texture_ 의 좋은 후보입니다. _array texture_ 들은 밉맵이나 _wrapping_ 한계를 고려할 때 2D 텍스쳐 아틀라스보다 더 적절합니다.

## single-precision floating 텍스쳐는 언제나 비쌉니다. (T6)

Textures with a floating-point format should be avoided whenever possible. If these textures are simply being used to represent a larger range of values, it may be possible to replace these with fixed point textures and scaling instructions. For instance, unsigned 16-bit integers cannot even accurately be represented by half-precision floats (FP16). These would have to be stored using single precision (FP32) leading to twice the memory and bandwidth requirements. It might be better to store these values in two components using 8 bits (LA8) and spend ALU instructions to unpack them in a shader.

단일 부동 소수점 포맷 텍스쳐는 가능하면 피하세요. 이런 텍스쳐가 큰 범위의 값을 표현하기 위해 보통 사용된다면, 이는 고정 소수점 텍스쳐와 스케일링 명령으로 바뀔 수 있습니다. 예를 들어 무부호 16-bit 정수는 심지어 반 부동 소수점 숫자로(FP16) 표현할 수 없습니다. 이를 단일 부동 소수점에 저장하는 것은 두배의 메모리와 대역폭을 필요로 하게 됩니다. 이는 8-bit 두 컴포넌트에 저장하고, 이를 쉐이더에서 언팩하는 계산 명령을 소모하는 것이 더 나을 것입니다.

Note: Floating-point textures may not support anything better than nearest filtering.

부동 소수점 텍스쳐는 _nearest filtering_ 보다 더 나은 것을 제공하지 않습니다.

## 거의 대부분의 경우 POT(power-of-two) 텍스쳐를 사용하십시오. (T7)

Although Non-Power-of-Two (NPOT) textures are supported in ES2 they come with a CLAMP_TO_EDGE restriction on the wrapping mode (unless relaxed by an extension). More importantly, they cannot be mipmapped (unless relaxed by an extension). For that reason, POT textures should be used when there is not significant memory and bandwidth to be saved from using NPOT. However, an NPOT texture may be padded internally to accommodate alignment restrictions in hardware and that the amount of memory saved might not be quite as large as the width and height suggests. As a rule of thumb, only large (i.e., hundreds of texels) NPOT textures will effectively save a significant amount of memory over POT textures.

2의 제곱이 아닌(NPOT) 텍스쳐는 ES2 에서 제공되지만, _wrapping mode_ 에서의 `CLAMP_TO_EDGE` 한계를 가집니다. 더 중요한 것은, 밉맵을 사용할 수 없습니다.(확장으로 늘어나지 않는다면) 이 이유 때문에 POT 텍스쳐는 NPOT 텍스쳐가 메모리와 대역폭을 줄일 수 없으므로, 사용되어야 합니다. 하지만 NPOT 텍스쳐는 하드웨어에서 _alignment_ 한계를 수용하기 위해 내부적으로 패딩되지만, width, height 를 더 늘려서 메모리의 양은 충분히 절약되지 못합니다. 경험적으로 오직 큰 NPOT 텍스쳐들이(즉, 수백개의 텍셀들) POT 텍스쳐 보다 의미있는 메모리 양을 아낄 수 있습니다.

## 텍스쳐 업데이트는 드물게 (T8)

Writing to GPU resources can be expensive—it applies to textures as well. If texture updates are required, then determine if they really need to be updated per frame or if the same texture can be reused for several frames. For environment maps, unless the lighting or the objects in the environment have been transformed (e.g., moved/rotated) sufficiently to invalidate the previous map, the visual difference may not be noticeable but the performance improvement can be. The same applies to the depth texture(s) used for shadow mapping algorithms.

GPU 리소스에 쓰기-텍스쳐에 저장하는 것은 비쌀 수 있습니다. 텍스쳐 업데이트가 필요하다면, 프레임별로 업데이트가 필요한지, 몇 프레임 동안 재사용이 가능한지 결정해야 합니다. 환경 맵의 경우, 이전의 맵이 무효화 되기에 충분할 만큼 라이팅이나 환경의 오브젝트들이 바뀌지 않은 경우, 시각적 차이는 크게 보이지 않지만, 성능 향상은 있을 수 있습니다. _shadow mapping_ 을 위한 깊이 텍스쳐도 같이 적용 가능합니다.

## 텍스쳐 업데이트는 효율적으로 (T9)

When updating an existing texture, use TexSubImage instead of re-defining its entire contents with TexImage when possible.

존재하는 텍스쳐를 업데이트할 때, 가능하다면 전체 내용을 `TexImage` 와 함께 재정의 하는 대신 `TexSubImage` 을 사용하세요.

Note: When using TexSubImage it is important to specify the same texture format and data type with which that the texture object was defined; otherwise, there may be an expensive conversion as texels are being updated.

`TexSubImage` 을 사용할 때, 정의된 텍스쳐 오브젝트와 같은 텍스쳐 포맷 및 데이터 타입을 명시하세요; 그렇지 않으면 업데이트 시 각 _texel_ 들을 비싸게 변환시켜야 합니다.

If the application is only updating from a sub-rectangle of pixels in client memory, then remember that the driver has no knowledge about the stride of pixels in your image. When the width of the image rectangle differs from the texture width, this normally requires a loop through single pixel rows calling TexSubImage repeatedly while updating the client memory offsets with pointer arithmetic. In this case, the unpack_subimage extension can be used (if supported) to set the UNPACK_ROW_LENGTH pixelstore parameter to update the entire region with one TexSubImage call.

만약 응용 프로그램이 클라이언트 메모리 내에서 부분 사각형의 픽셀들로부터 업데이트 해야 한다면, 드라이버는 당신의 이미지 내의 픽셀의 스트라이드를 모른다는 것을 기억하세요. 이미지 사각형의 넓이가 텍스쳐 크기에 따라 달라질 때, 보통 한 행별로 클라이언트 메모리에서 포인터 연산을 통해 _TexSubImage_ 를 호출하는 것을 반복적으로 루프에서 처리해줘야 합니다. 이 경우 _unpack_subimage_ 확장을(지원한다면) 한번의 `TexSubImage` 호출과 함께 전체 영역을 업데이트 하기 위해 `UNPACK_ROW_LENGTH` 픽셀저장 파라미터를 설정하여 사용할 수 있습니다. 

## 각각의 알파 테스팅/블렌딩 렌더링을 다른 렌더링과 분리하십시오. (T10)

It is sometimes possible to improve performance by splitting up the rendering based on whether alpha blending or alpha testing is required. Use separate draw calls for opaque geometry so these can be rendered with maximum efficiency with blending disabled. Perform other draw calls for transparent geometry with alpha blending enabled, taking into account the draw ordering that transparent geometry requires. As always, benchmarks should be run to determine if this improves or reduces the frame rate since you will be batching less by splitting up draw calls.

알파 블렌딩이나 알파 테스팅이 필요한지 아닌지에 따라 렌더링을 분리하는 것은 성능 향상의 가능성을 가지고 있습니다. 불투명 오브젝트를 위해 드로우콜을 분리하세요, 그러면 블렌딩이 비활성화 되어 있어 효율적으로 렌더링할 수 있습니다. 알파 블렌딩이 켜진 상태에서 다른 투명 오브젝트의 드로우콜을 실행하고, 필요한 투명 오브젝트의 정렬을 고려하세요. 항상 그렇듯이, 이는 drawcall 의 batching 을 줄이므로, 이 방법이 _frame rate_ 를 증가시킬지, 감소시키는지 결정하기 위해 벤치마크를 하세요.

## 적절하게 텍스쳐 필터링 하십시오. (T11)

Do not automatically set expensive texture filters and enable anisotropic filtering. Remember that nearest-neighbor filtering always fetches one texel, bilinear filtering fetches up to four texels and trilinear fetches up to eight texels. However, it can be incorrect to draw assumptions about the performance cost based on this. Bilinear filtering may not cost four times as much as nearest filtering, and trilinear can be more or less than twice as expensive as bilinear. Even though textures have mipmaps, it does not automatically mean trilinear filtering should be used. That decision should be made entirely from observing the images from the running application. Only then can a judgment be made if any abrupt changes between mipmap levels are visually disturbing enough to justify the cost of interpolating the mipmaps with trilinear filtering. The same applies to anisotropic filtering, which is significantly more expensive and bandwidth intensive than bilinear or trilinear filtering. If the angle between the textured primitives and the projection plane (e.g. near plane) is never very large, there is nothing to be gained from sampling anisotrophically and there is potentially lower performance. Therefore, an application should start off with the simplest possible texture filtering and only enable more expensive filtering after users have inspected the output images. It might be worthwhile to benchmark the changes and take notes along the way. This will provide a better indication of the relative cost of filtering method and if concessions must be made if the performance budget is exceeded.

자동으로 비싼 텍스쳐 필터를 설정하지 말고, _anisotropic filtering_ 을 활성화 하세요. _nearest-neighbor filtering_ 은 하나의 텍셀, _bilinear filtering_ 은 4개의 텍셀, _trilinear filtering_ 은 8개의 텍셀을 가져오는 것을 명심하세요. 하지만 이를 기반으로 성능 비용에 대한 가정을 세우는건 부정확합니다. _bilinear filtering_ 은 _nearist filtering_  처럼 4배만큼 비용이 들지 않으며, _trilinear filtering_ 은 _bilinear filtering_ 의 두배 내외의 비용을 가질 수 있습니다. 텍스쳐가 밉맵이 있다면, 자동으로 _trilinear filtering_ 을 사용하는 것을 의미하는 것은 아닙니다. 이 결정은 실행중인 응용프로그램에서부터 이미지를 관찰하며 전체적으로 만들어져야 합니다. _anisotropic filtering_ 또한 같이 적용되며, 이는 _bilinear_, _trilinear_ 보다 더 비싸고 대역폭도 많이 차지합니다. 텍스쳐화된 도형과 카메라 평면의 각도가 매우 크다면, 비등방성(_anisotropic_) 샘플링에서는 아무것도 얻을 수 없고 낮은 성능을 가질 수 있습니다. 그러므로, 응용 프로그램은 처음에는 가능한 제일 간단한 필터링으로 시작하여,사용자가 더 나은 결과 이미지를 보려한 후에 더 비싼 필터링을 활성화하세요. 변화에 따라 기록하며 벤치마크하는 것은 의미있을 것입니다. 

## 텍스쳐 타일링을 활용하려 노력하십시오. (T12)

It is common for images to contain the same repeated pattern of pixels. Or an image might repeat a few patterns that are close enough in similarity that they could be replaced with a single pattern without impacting image quality. Tiling textures saves on memory and bandwidth. Some image processing applications can identify repeated patterns and can crop them so they can be tiled infinitely without seams when using textures with the REPEAT wrap mode. Sometimes even a quarter of a tile or shingle may be sufficient to store while using MIRRORED_REPEAT. Consider if tiling variation can be restored or achieved with multi-texturing, using for instance a less expensive grey-scale texture that repeats at a different frequency to modulate the texels from the tiled texture.

같은 반복되는 패턴의 픽셀을 가진 이미지는 흔합니다. 이미지 퀄리티에 영향을 끼치지 않고 하나의 패턴으로 바꿀만한 충분한 유사성을 가진 몇 패턴을 반복할 수도 있습니다. 텍스쳐 타일링은 메모리와 대역폭을 아낄 수 있습니다. 몇몇 이미지 처리 응용 프로그램은 반복되는 패턴을 인지하고 버릴 수도 있습니다. _REPEAT_ _wrap mode_ 의 텍스쳐를 사용할 때 _seam_ 없이 무한한 타일링이 가능합니다. 심지어 가끔 타일 혹은 지붕 덮개를 _MIRRORED_REPEAT_ 를 사용하고, 4분의 1만 저장해도 충분할 수도 있습니다. 여러가지 타일링 방법이 여러 텍스쳐와 함께 처리될 수 있는지 고려하세요. 예를들어 덜 비싼 그레이스케일 텍스쳐를 타일링된 텍스쳐와 다른 주파수를 통해 섞을 수도 잇습니다.

## 프레임 버퍼 오브젝트를(FBO) 동적으로 생성되는 텍스쳐를 위해 사용하십시오. (T13)

OpenGL ES comes with functions for copying previously rendered pixels from a framebuffer into a texture (TexCopyImage, TexCopySubImage). These functions should be avoided whenever possible for performance reasons. It is better to bind a framebuffer object with a texture attachment and render directly to the texture. Make sure you check for framebuffer completeness.

OpenGL ES 는 프레임버퍼에서 이전에 렌더링된 픽셀을 텍스쳐에 복사하는 기능을 제공합니다. (_TexCopyImage_, _TexCopySubImage_) 이 기능들은 성능상의 이유로 피해야 합니다. 프레임 버퍼 오브젝트를 참조할 텍스쳐로 사용하여(_texture attachment_) 직접 텍스쳐에 렌더링 하는 것이 더 낫습니다. 프레임버퍼 완전성을 확인하세요.

Note: Not all pixel formats are color-renderable. Formats with 3 or 4 components in 16 or 32 bits are color-renderable in OpenGL ES 2.0, but LUMINANCE and/or ALPHA may require a fall-back to TexCopyImage functions.

참조: 모든 픽셀이 색 렌더링이 가능한건 아닙니다. 16-bit, 32-bit 컴포넌트를 3,4 개 가진 포맷은 OpenGL ES 2.0 에서 색 렌더링이 가능하지만, _LUMINANCE_ 와 _ALPHA_ 는 _TexCopyImage_ 함수를 사용하는 것을 대비해야 합니다.

# 기타

This topic contains miscellaneous OpenGL ES programming tips.

해당 주제는 기타 OpenGL ES 프로그래밍 팁을 담고 잇습니다.

## FBO 내용을 다시 읽는 것을 피하십시오. (M1)

Reading back the framebuffer flushes the GL pipeline and limits the amount CPU/GPU parallelism. Reading frequently or in the middle of a frame stalls the GPU and limits the throughput with lower frame rate as a result. If the buffer contents must be read back (perhaps for picking 3D objects in a complex scene), it should be done minimally and scheduled at the beginning of the next frame. In the special case that the application is reading back into a sub-rectangle of pixels in client memory, the pack_subimage extension (if supported) is very useful. Setting the PACK_ROW_LENGTH pixel store parameter will reduce the loop overhead that will otherwise be necessary (T9).

프레임버퍼를 다시 읽는 것은 _GL pipeline_ 을 비우며, 많은 CPU/GPU 병렬성을 제한합니다. 자주 혹은 프레임 처리 중간에서 읽는 것은 GPU 의 _stall_ 을 야기하고, 처리량을 제한시켜 결과적으론 낮은 _frame rate_ 를 가진다. 버퍼 내용이 반드시 다시 읽혀야 한다면(만약에 복잡한 환경에서 3D 오브젝트를 피킹해야 한다면), 최소한으로 실행되고, 다음 프레임의 시작에서 되도록 스케쥴링 해야 합니다. 응용 프로그램이 부분 사각형 내의 픽셀을 클라이언트 메모리로 읽어와야 하는 특별한 경우, _pack_subimage_ 확장(지원하면)은 유용합니다. _PACK_ROW_LENGTH_ 파라미터를 설정하여 루프 오버헤드를 필요한 만큼으로 줄일 수 있습니다. (t9)

## 필요없는 버퍼 청소를 피하십시오. (M2)

If the application always covers the entire color buffer for each frame, then bandwidth can be saved by not clearing it. It is a common mistake to call Clear(GL_COLOR_BUFFER_BIT) when it is not necessary. If only part of the color buffer is modified, then constrain pixel operations to that region by enabling scissor testing and define a minimal scissor box for the region. The same applies to depth and stencil buffers if full screen testing is not needed.

응용 프로그램이 항상 각 프레임별로 전체 색 버퍼를 처리하는 경우, 비우지 않음으로써 대역폭을 아낄 수 있습니다. 이는 필요하지 않아도 청소하는(GL_COLOR_BUFFET_BIT) 일반적인 실수입니다. 색 버퍼의 일부만 수정되었다면, _scissor testing_ 을 활성화하여 픽셀 연산을 제한시키고, 최소한의 _scissor box_ 를 정의하세요. 전체화면 테스트가 필요하지 않은 경우 깊이 버퍼와 스텐실 버퍼 또한 마찬가지입니다.

## 필요하지 않은 블렌딩은 비활성화 하십시오. (M3)

Most blending operations require a read and a write to the framebuffer.

대부분의 블렌딩 연산은 프레임버퍼 읽기/쓰기 가 필요합니다.

Note: Memory bandwidth is often doubled when rendering with blending is enabled. The number of blended fragments should be kept to a minimum—it can drastically speed up the GL application.

보통 메모리 대역폭은 블렌딩이 활성화 된 경우 두배로 커집니다. 블렌드된 _fragment_의 숫자를 최소한으로 유지키세요-이는 GL 응용 프로그램의 엄청난 속도 향상이 가능합니다.

## 메모리 파편화를 최소화 하십시오. (M4)

Buffer objects and glTexImage* functions are effectively graphics memory allocations. Reusing existing buffer objects and texture objects will reduce memory fragmentation. If geometry or textures are generated dynamically, the application should allocate a minimal pool of objects for this purpose during application initialization. It may be that two buffers or textures used in a round-robin fashion are optimal for reducing the risk that the GPU is waiting on the resource. Also, recall that sampling a texture that is being rendered to, at the same time, is undefined. This can be another reason to alternate between objects. For more information, see Memory Fragmentation in this appendix.

버퍼 오브젝트와 `glTexImage*` 류의 함수는 효과적인 그래픽 메모리 할당입니다. 존재하는 버퍼 오브젝트 및 텍스쳐 오브젝트를 재사용하여 메모리 파편화를 줄일 수 있습니다. 만약 기하 혹은 텍스쳐가 동적으로 만들어진다면, 응용 프로그램은 초기화 동안 최소한의 오브젝트 풀을 할당합니다. 이는 두 버퍼와 텍스쳐를 라운드로빈 방법으로 사용하여 GPU가 리소스를 기다리는 리스크를 줄이는 최적의 방법입니다. 또한 텍스쳐를 동시에 샘플링 하는 것은 정의되지 않았습니다. 이는 오브젝트 간의 대체제를 사용할 다른 이유입니다. 더 많은 정보는 Appendix 의 메모리 파편화 항목을 보세요.

# OpenGL ES 최적화 하기

Optimization is an iterative process. It can be time consuming, especially without prior experience determining where bottlenecks tend to occur. Effort should be directed towards the critical areas instead of starting a random place in the rendering code. When the graphics application is complex it may be difficult to know where to start or exactly where optimizations will yield the best return.

## 관리가능한 청크로 분석 나누기 Partition the analysis into manageable chunks

Many rendering applications are complex and consist of hundreds of objects. But usually they consist of logically separate rendering code. For example, a rendered image may consist of roads, buildings, landmarks, points of interest, sky, clouds, buildings, water, terrain, icons, and a 2D user interface. It is helpful to write the GL application such that rendering of each type of object can be disabled easily. This allows easy identification of the most expensive objects when benchmarking and therefore makes optimizing the rendering code more manageable.

## 그래픽 파이프라임의 병목에 익숙해지기

It is important to begin optimizations by identifying the performance bottlenecks at the different stages in the graphics pipeline. Because the work introduced in the beginning of the pipeline normally affects the work needed at later stages, it often makes sense to work backwards from the end of the pipeline. An introduction to identifying graphics bottlenecks can be found in the GPU Gems book, "Chapter 28. Graphics Pipeline Performance" (Cem Cebenoyan, NVIDIA).

# 메모리 파편화 피하기

Memory Fragmentation generally is a bad thing. This is especially true for computer graphics applications. In addition to avoiding system memory fragmentation, a graphics application should strive to avoid video memory fragmentation as well.
Fortunately, controlling video memory fragmentation has techniques very similar to those used to avoid system memory fragmentation. Since system memory fragmentation control is fairly well known, this document will only treat system memory issues in passing and focuses on video memory techniques.

# 비디오 메모리 개괄

Video memory is much more heterogenous than system memory.
NVIDIA video memory allocation algorithms have to take the following into account:
- There are multiple types of video memory types. The number and names of the types vary by GPU model, but GPUs generally have at least two; linear, which is essentially unformatted, and one or more GPU specific types. The GPU tracks different types of memory, and will access and write them differently. The types are important because GPU native types can be faster for a given set of operations; in some GPU architectures, the difference is small, on the order of 10-15%. On others, it can be quite large, more than 100% faster than linear memory.
- Video memory is often banked, especially for mipmapped textures. In most architecture, alternating mipmap levels for a given texture must be put in separate banks. This separation is mandatory in most NVIDIA GPUs.
- In addition to the restrictions above, different memory regions have different alignment restrictions, to match host pages, improve DMA performance, or speed up framebuffer scan out. These alignment requirements may be orthogonal to the memory types, adding further complication.
- The allocator may have other special restrictions that enhance performance, such as distributing allocations to a sequence of different banks to improve allocation speed.
- These extra constraints complicate the video memory allocator, and make allocations much more sensitive to reductions in available video memory. This is the major reason why NVIDIA does not support multiple independent heaps in video memory, instead requiring the application to allocate in such a way as to minimize fragmentation.

# 비디오 메모리 할당/해제

This topic describes considerations for allocating and freeing video memory.

## 버퍼 할당

When using OpenGL ES/EGL, there is only a small set of APIs that actually lead to long-term video memory buffer allocation:

```
glBufferData(enum target, sizeiptr size, const void *data, enum usage)
 
glTexImage2D(enum target, int level, int internalFormat, sizei width, sizei height, int border, enum format, enum type, const void *pixels)
 
glCopyTexImage2D(enum target, int level, enum internalformat, int x, int y, sizei width, sizei height, int border)
```

Note: The glCopyTexImage2D function allocates only when it copies to a null.

```
eglCreateWindowSurface(EGLDisplay dpy, EGLConfig config, NativeWindowType win, const EGLint *attrib_list)
 
eglCreatePbufferSurface(EGLDisplay dpy, EGLConfig config, const EGLint *attrib_list)
 
eglCreatePixmapSurface(EGLDisplay dpy, EGLConfig config, NativePixmapType pixmap, const EGLint *attrib_list)
```

## 버퍼 해제

A similar set of APIs free allocated video memory buffers, whether they are textures, VBOs, or surfaces:

```
glDeleteBuffers(sizei n, uint *buffers)
 
glDeleteTextures(sizei n const uint *textures)
 
eglDestroySurface(EGLDisplay dpy, EGLSurface surface)
```

Note: The glDeleteTextures function works only if the texture object is greater than zero; the default texture can't be explicitly deleted (although it can be replaced with a texture containing one or two dimensions of zero, which accomplishes the same thing).
Conceptually, these calls can be thought of as malloc() and free() for VBOs and texture maps, respectively. The same techniques for avoiding fragmentation can also be applied.

## 버퍼의 부분지역(subregion) 업데이트

In many cases, avoiding fragmentation means placing multiple objects into the same shared buffer, or reusing a buffer by deleting or overwriting an older object with a newer one. OpenGL ES provides a method for updating an arbitrary section of allocated VBOs, textures, and surfaces:

```
glBufferSubData(enum target, intptr offset, sizeiptr size, const void *data, enum usage)
 
glTexSubImage2D(enum target, int level, int xoffset, int yoffset, sizei width, sizei height, enum format, enum type, const void *pixels)
 
glCopyTexSubImage2D(enum target, int level, int xoffset, int yoffset, int x, int y, sizei width, sizei height)
 
glScissor(int left, int bottom, sizei width, sizei height);
 
glViewport(int x, int y, sizei w, sizei h)
```

The glTexSubImage2D and glCopyTexSubImage2D function update a subregion of the target texture image. In the first case, the source comes from an application buffer; in the second, from a rendering surface.
The glScissor and glViewport functions limit rendering to a subregion of a rendering surface. The first specifies the region of the buffer that the glClear function will affect; the second updates the transforms to limit rendered OpenGL ES primitives to the specified subregion.

## 버퍼의 부분지역(subregion) 사용

Completing the functionality needed to reuse allocated buffers is the ability to use an arbitrary subregion of a texture, VBO, or surface:

```
glDrawArrays(enum mode, int first, sizei count)
 
glReadPixels(int x, int y, sizei width, sizei height, enum format, enum type, void *data)
 
glCopyTexImage2D(enum target, int level, int x, int y, sizei width, sizei height)
 
glCopyTexSubImage2D(enum target, int level, int xoffset, int yoffset, int x, int y, sizei width, sizei height)
```

For VBOs, the glDrawArrays function allows the application to choose a contiguous subset of a VBO.
For textures, there's no explicit call to limit texturing source to a particular subregion. But texture coordinates and wrapping modes can be specified in order to render an arbitrary subregion of texture object.
For surfaces, the glReadPixels function can be used to read from a subregion of a display, when copying data back to an application-allocated buffer.

The glCopyTexImage2D and glCopyTexSubImage2D functions also restrict themselves to copying from a subregion of the display surface when transferring data to a texture map. The only area that's problematic is controlling the direct display of window surface back buffer. OpenGL ES and EGL have no way to show only a subregion of the backbuffer surface, but the native windowing systems may have this functionality.

# 비디오 메모리 관리 좋은 예시 

The following is a list of good practices when allocating video memory to avoid or minimize fragmentation:

## 1. 큰 버퍼를 미리 할당하기

Ideally allocate large buffers at the start of the program. On average, allocating large surfaces gets more difficult as more allocations occur. When more allocations occur, free space is broken into smaller pieces.

## 2. 수많은 적은 메모리 할당을 적은 갯수의 많은 할당으로 합치기 

Small allocations can disproportionately reduce available free space. Not only does the allocator have a fixed overhead per allocation, regardless of size, but small allocations tend to break up large areas of free space into smaller pieces.
The ability to load a subregion of a VBO or texture map, and the ability to render that subregion independently, makes it possible to combine VBOs and textures together. For textures, a large texture can be used to hold a grid of smaller images. For VBOs, multiple vertex arrays can be combined end to end into a larger one. Besides reducing fragmentation, combining related images into a single texture, and related vertex arrays into a single VBO often improves rendering time, since it reduces the number of glBindBuffer or glBindTexture calls required to render a set of related objects.

## 3. 할당된 버퍼 사이즈의 분포를 줄이고, 이상적으로 하나의 사이즈로 만들기

Allocating buffers of varying sizes, especially sizes that aren't small multiples of each other, is disruptive of memory space and causes fragmentation. The ability to load and render a subset of a VBO or texture means that data loaded doesn't have to match the size of the allocated buffer; as long as it's smaller, it will work.
This approach does waste space, in that some of the allocated buffer isn't used, but this waste is often offset by the saving in reduced fragmentation and fixed allocation overhead. This approach can often be combined with approach (2) (combining multiple objects into one allocated buffer) to reduce total wastage. Generally, it's safe to ignore wastage it it's a small percentage of the allocated buffer size (say < 5%).
This approach is particularly good for dynamically allocated data, since fixed size blocks can often be freed and reallocated with little or no fragmentation. If wastage is excessive, a set of buffer sizes can be chosen (often a consecutive set of power of two sizes), and the smallest free buffer that will contain the data is used.

## 4. 가능한 버퍼를 해제/재할당 하지말고 재사용하기

The ability to reload a previously allocated buffer with new data for both VBOs and textures makes this technique practical. Reuse is particularly important for large buffers; it is often better to create a large buffer at program start, and reuse it during mode switches, etc., even if it requires allocating a larger buffer to handle all possible cases.

## 5. 동적할당 최소화 하기

If possible, take memory allocation and freeing out of the inner loop of your application. The ability to reuse buffers makes this practical, even for algorithms that require dynamic allocation. Even with reuse, however, it's still better to organize the code to minimize allocations and frees, and to move the remaining ones out of the main code path as much as possible.

## 6. 여러 동적 할당을 묶으려 노력하기

If dynamic allocation is mandatory, try to group similar allocations and frees together. Ideally, an allocation of a buffer is followed by freeing it before another allocation happens. This rarely can be done in practice, but combining a group of related allocations is often nearly as effective.
Again, allocations and frees should be replaced whenever possible. Grouping them is a last resort.

# 그래픽 드라이버 CPU 사용량

In some cases, the reported graphics driver CPU usage may be high, but in fact the yield is related to other CPUs. To reduce the reported CPU usage, set the environment variable as follows:

```
$ export __GL_YIELD=USLEEP

```

# 성능 가이드라인

The NVIDIA Tegra system on a chip (SOC) includes an extremely powerful and flexible 3D GPU whose power is well matched to the OpenGL ES APIs. For basic guidelines and tips on optimal content rendering, see [OpenGL ES Performance for the Tegra Series Guidelines.](https://docs.nvidia.com/drive/drive_os_5.1.6.1L/nvvib_docs/DRIVE_OS_Linux_SDK_Development_Guide/baggage/tegra_gles2_performance.pdf)
