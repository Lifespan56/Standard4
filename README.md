분석 문제 : 분석한 내용을 직접 작성하고, 강의의 코드를 다시 한번 작성하며 복습해봅시다.

Equipment와 EquipTool 기능의 구조와 핵심 로직을 분석해보세요.

Equipment.cs

스크립트 명으로 직관적으로 알 수 있는 기능 장착품에 대한 처리

칼을 장착했다고 예를 들면

장착한 이 칼이 카메라에 보이기 위한 처리가 우선 필요

애니메이션이 있다면 이 칼의 애니메이터도 필요

사용자의 입력에 따라 무엇인가 수행되어야 한다면 입력을 받아오는 인풋

혹은 구독방식을 사용하려면 발행자에 대한 참조가 필요

이 칼 자체의 스탯정보도 필요

라고 생각했지만

현재 장착아이템을 참조할 curEquip

curEquip의 부모Transform equipParnet

입력을 받아올playerController 3가지만 필요했다



기본적으로 카메라가 2가지이다

일반적으로 생각하는 카메라 기능을 하는 Main

장착한 아이템만 컬링마스크를 이용해 찍는 EquipCamera

equipParnet는 편의상 하이어라키에서 장비아이템을

EquipCamera의 자식화를 하기위해 사용한다



앞서 분석한 UI_Inventory에서 장착하기를 누르면

이 스크립트의 Equip함수에 아이템SO를 매개변수로 넘겨준다

현재장비템 curEquip이 null이든 아니든 일단 장착해제 함수를 수행한다

curEquip != null이면 curEquip.gameObject를 파괴하고 curEquip를 비우는 처리밖에 없다

다시 돌아와 비어있는 curEquip에 매개변수로 받아온 아이템SO를 이용해

동적생성한다 아이템SO는 게임화면에서 보여줄 모델프리팹을 가지고 있다

curEquip = Instantiate(itemBasicSO.forEquipPrefab, equipParnet).GetComponent<Equip>();

이용자의 입력을 받아 처리하는 함수도 가지고있다

OnAttackInput(InputAction.CallbackContext context)

누른 상태고 curEquip가 null이 아니고 플레이어가 인벤을 보는중이 아니면

curEquip.OnAttackInput();

curEquip는 Equip형이고 Equip.cs자체는 

구현이 되어있지 않은 함수 virtual void OnAttackInput()만 가지고 있다

Equip를 상속받은 무엇인가가 실질적인 출력행동을 할것이다



Equipment 기능의 구조

사용자 입장에서의 상식적인 아이템 탈, 부착

컬링 마스크를 사용하는 EquipCam에 보여줄 비쥬얼세팅

이용자로부터 입력받기



Equipment 핵심로직

인벤토리 장착버튼으로 부터 넘겨받은 아이템SO를 통한 객체생성



EquipTool.cs

Equipment.cs에서 스크립명만 보고 직관적으로 분석한 기능이 여기서 쓰인다

장착아이템이 가져야할 모든 스탯정보, 애니메이터

장비와 장비도구.. 뭔 차이지?

EquipTool은 도구 그 자체

Equipment는 플레이어가 장착한 것

그래서 내가 PlayerEquipment로 이름을 바꿔놓은 듯 하다



Equip를 상속받고

override void OnAttackInput()하고있다

OnAttackInput() 함수

bool변수를 통해 입력을 받은 후 제어하고있다

기본적으로 플레이어가 버튼을 입력하면 장착한 장비를 휘두른다

장비는 휘두르는 딜레이를 가지고 있고 휘두르는 애니메이션도 가지고있다

OnAttackInput()함수에 진입하면 조건문으로 !isAttacking일때만
기능하도록 제어되어있다

isAttacking = true,
휘두르는 애니메이션재생

Invoke("함수",딜레이)로 장비 딜레이만큼 기다린 후

isAttacking를 =false로 만드는 함수를 실행해
한번 휘두르면 딜레이 만큼 지나야 다시 휘두를 수 있도록 구성되어있다



OnHit() 함수

위의 함수가 동작제어라면 이 함수는 실질적인 데이터 변동

무기를 휘둘렀을 때 레이케스트를 통해 현재 장비의 유형과 대상의 유형을 비교, 판단해
자원이 채취될 것인지 상대에게 데미지를 입힐 것인지를 결정한다

이 함수 자체는 애니메이션 클립으로 호출된다

장비를 휘두르는 애니메이션의 구성은 크게

들어올리기, 앞으로 휘두르기, 돌아오기 일것이다

사용자로부터 입력을 받으면 들어올리기부터 시작인데 실제로 데미지를 입는 등의 처리는

앞으로 휘둘러 대상에게 비쥬얼적으로 접촉 했을때 일것이다

앞으로 휘두르는 애니메이션 키프레임에 맞춰 클립을 사용해 이 함수가 호출된다

시점이 1인칭 이기때문에 앞의 대상의 정보를 가져오기위한 레이케스트는

Ray ray = cam.ScreenPointToRay(new Vector3(Screen.width / 2, Screen.height / 2, 0));
RaycastHit hit;
if (Physics.Raycast(ray, out hit, effectiveRange))

레이케스트의 시작점, 반환받을 정보, 레이케스트의 길이



EquipTool.cs 기능의 구조

사용자 입력 > OnAttackInput() > bool조건판단 > 애니메이션 재생 > bool제어

애니메이션 중간 클립 > OnHit() > Raycast > 현재장비유형과 RaycastHit의 유형으로 판단, 처리



EquipTool.cs 핵심 로직

OnHit().cs의 호출시점, 사용자의 화면 시점에따른 Raycast 원점 설정과 RaycastHit






Resource 기능의 구조와 핵심 로직을 분석해보세요.



Resource.cs

사용자가 채취 할수있는 자원

강의에서는 나무토막을 얻을 수 있는 나무 한그루

나무는 맞았을 때 나무토막을 뱉어줘야한다

드랍할 아이템의 정보인 SO를 가지고 있어야한다

이 SO는 드랍용 프리팹을 가지고 있다

한번에 몇개를 드랍 할 것인지

총 몇개를 드랍 할 것인지

3가지 필드와 Gather(Vector3 hitPoint,Vector3 hitNormal)함수만 존재한다

Gather()는 위의 EquipTool.cs의 OnHit()에서 유형판단 후 호출된다



사실상 Resource.cs는 중계역할로 보인다

사용자의 장비에 의해 함수가 호출되면 현재 용량을 감소시키고

ItemSO의 프리팹을 이용해 채취량만큼 동적생성하고 뱉어준다

드랍위치는 도구에서 호출시에 넘겨받는

RaycastHit hit ;

hit.point, hit.normal
그냥 RaycastHit에 맞은 물체의 포지션 정보같은 것도 담겨있구나 정도의 이해





Resource.cs 기능의 구조

외부에서 Gather()호출 > 남은 용량있는지 판단 > ItemSO의 프리팹을 동적생성



Resource.cs 핵심 로직

남은 용량이 있는가의 판단정도

사실상 호출시점, ItemSO가 더 중요하지 않나 싶음




분석 문제 : 분석한 내용을 직접 작성하고, 강의의 코드를 다시 한번 작성하며 복습해봅시다.

AI 네비게이션 시스템에서 가장 핵심이 되는 개념에 대해 복습해보세요.

NPC 기능의 구조와 핵심 로직을 분석해보세요.



NPC.cs

강의에서는 적에 해당하는 곰이 가지고있다

기본적으로 가져야할 체력,속도,공격력 등의 스텟은 제외하고

AINav기능을 활용한 배회, 추적이 핵심일 것이다



유니티에서 기본제공하는 navMeshAgents 컴포넌트

복잡한 알고리즘 작성없이 장애물을 피하고, 도보 가능한 길만을 구분해 알아서 가게해주는

딸깍이다



필드변수

우선 방황기능에 사용하기위한 방황 한번의 최소, 최대거리

Idle상태에서 다시 방황하기까지의 최소, 최대대기시간

감지거리

감지범위 시야각

플레이어와의 거리
피격시 점멸을 해주기위한 SkinnedMeshRenderer (3D판 spriteRenderer)

모델의 비쥬얼적 행동을 위한 Animator
이 NPC의 상태 열거형 aiState

그리고 navMeshAgents

navMeshAgents는 유니티 기본제공 컴포넌트이다

인스펙터창으로 보는게 빠르다 속도, 회전속도, 가속도 등등

기존의 스크립트 작성으로 AddForce하는 것이 그냥 변수만 입력하면 알아서 움직인다

중요한 것은 어디로 갈것인지 목적지를 설정해주는 것이다

보다보면 개꿀 기능이 많다 Vector3.Distance라던지 NavMesh.SamplePosition라던지

몰랐다면 수동으로 x,y,z다 계산했을 것이다

NavMesh.SamplePosition같은 경우는 주변 반경으로 뭔가를 감지할때 사용된다

Agent가 갈수있는 지형인지 bool로 반환하는 함수라고한다

do while로 해보라고 강의에 있었던 기억이 있었고

agent중심 반경으로 지름이랜덤인 구를 생성해 그곳으로 agent가 갈수있는가를 판단해 보내는 로직이다

여지껏 알고있던 한 지점을 콕 찝어서 목표로 설정하는것과 괴리감이 생긴다

그냥 구를 만들고 그 구에 감지되는 수많은 곳중에 어디라는 소리인가에 대한 원초적인 의문이 들었다

디버그 레이를 찍어 씬뷰에서 확인해봤다 땅 밑으로 레이가 들어가긴 하지만 레이의 방향과 이동방향이 같은 것 처럼 보인다

NavMesh.SamplePosition함수 자체가 아니라 Random.onUnitSphere에 답이 있었다

내가 원하는 해답인 구 형태 중에서 특정한 지점자체가 Random.onUnitSphere이었다

private Vector3 GetWanderLocation()
{
    NavMeshHit hit;
    //기준점(scale1짜리 구*랜덤범위),
    int i = 0;

    do
    {
        sourcePosition = transform.position + (Random.onUnitSphere * Random.Range(minWanderDistance, maxWanderDistance));
        
        NavMesh.SamplePosition(sourcePosition, out hit, maxWanderDistance, NavMesh.AllAreas);

        Vector3 dir = sourcePosition - transform.position;

        Debug.DrawRay(transform.position, dir, Color.red, 10f);

        i++;
        
        if (i == 30) break;
    } 
    while (Vector3.Distance(transform.position, hit.position) < detectDistance);

    return hit.position;
}
