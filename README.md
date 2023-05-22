# redesigned-invention
최적화

AOS류 게임 제작에 관한 기능 구현 및 최적화에 대한 파일 입니다

1.움직임 구현
   [
     게임내에서 움직임을 구현해야 하는 요소들은 많이 존재한다. 하지만 플레이어등을 포함한 모든 오브젝트에 관한 움직임 구현말고, 동적으로 확인이 바로 가능한 것에 대한 움직음을 구현하는 것에 대해 보여줄것이다.
     
     일단 움직음을 구현하려면 여러 방법을 사용할 수 있다.
     예를 들어 addforce혹은 transform.positon을 startcoroutine 방식으로 움직이는 방법 등이있다.
     
     하지만 이번 프로젝트에서는 NavMeshAgent를 사용하여 움직임을 구현하는걸 보여줄 것이다.
     
     NavMeshAgent를 길라잡이라고 생각하면 편하게 이해할수 있다.
     
     Player,Monster,Npc이렇게 3종류에 대한 움직임을 구현하는 법
     
     -Player : 1.키보드의 입력을 받아 움직이게 한다
               2.모니터에 마우스를 클릭하여 움직임을 구현한다.
               
               이번 프로젝트에서는 후자를 선택할것이다.
               
               마우스 클릭을 이용한 움직을 구현하기 위해서는 몇가지의 요소들이 필요하다
                 1.NavMeshAgent
                 2.Camera와 플레이어의 상관 관계
                 3.OriDirection =플레이어의 원래 방향 정보를 가지고 있는 Vector3값
                 4.currentV,currentH 중력 대신 사용할 변수들
                 
                 일단은 이렇게 4가지만 으로 움직임이 구현이 가능하다.
                 [코드는 NplayerMove의 DefaultMove라고 쓰여져있는 cs줄에 포함이 되어있다.]
                
    
    -Monster : 플레이어가 일정 범위에서 발견되었을 때 state값을 다르게 하여서 움직임 구현 
               1.플레이어가 감지 범위 밖에 있는 경우
               2.플레이어가 감지 범위 안에 있을 경우
               3.플레이어가 공격 범위 안에 들어와있을 경우 
               
               몬스터의 state는 EnumData라는   Cs로 따로 관리
               public enum MonsterState
               { 
               IDLE =0,
               TRACE,
               ATTACK,
               DEAD,
               }
               
               몬스터의 움직임은 매 플레임마다 state값을 확인하여 구동해야하므로 LateUpdate에서 CheckMonsterStata로 구동시킨다
               -조건은 플레이어와의 거리로 확인
               [단, 플레이어와의 움직음을 구현하기에 앞서 나의 플레이어를 찾아야하므로, 플레이어를 PunManager에 대입한다->Nplayer.Cs에 있음]
               
               몬스터의 공격같은경우, collider를 충돌시키는 것보다 플레이어가 게임을 하는데 용이하게 하기 위해 Animation에 Event를 추가하여 
               공격 타이밍에 플레이어가 있을 경우 그 플레이어에게 데미지를 넣는 식으로 구현
               
                private void OnDrawGizmos()
                {
                    //체크를 위해 따로 만들어놓은 bool
                    if (!DebugMode)
                        return;

                    Vector3 myPos = transform.position + Vector3.up * 0.5f;
                    Gizmos.DrawWireSphere(myPos, viewRadious);

                    float lookingAngle = transform.eulerAngles.y;
                    Vector3 rightDIr = AngleToDir(transform.eulerAngles.y + viewAngle * 0.5f);
                    Vector3 leftDIr = AngleToDir(transform.eulerAngles.y - viewAngle * 0.5f);
                    Vector3 lookDIr = AngleToDir(lookingAngle);

                    Debug.DrawRay(myPos, rightDIr * viewRadious, Color.blue);
                    Debug.DrawRay(myPos, leftDIr * viewRadious, Color.blue);
                    Debug.DrawRay(myPos, lookDIr * viewRadious, Color.cyan);

                    HitTargets.Clear();
                    Collider[] Targets = Physics.OverlapSphere(myPos, viewRadious, targetMask);

                    if (Targets.Length == 0)
                        return;

                    foreach(Collider EnemyColl in Targets)
                    {
                        Vector3 targetPos = EnemyColl.transform.position;
                        Vector3 targetDir = (targetPos = myPos).normalized;
                        //두 벡터사이의 각을 Mathf.Acos을 통해 라디안으로 바꾸고, 그 값에 Mathf.Rad2Deg를 곱하여 각으로 바꾼다.
                        float targetAngle = Mathf.Acos(Vector3.Dot(lookDIr, targetDir)) * Mathf.Rad2Deg;
                        if(targetAngle <= viewAngle *0.5f &&!Physics.Raycast(myPos,targetDir,viewRadious,obstacleMask))
                        {
                            HitTargets.Add(EnemyColl.gameObject);
                            if (DebugMode)
                                Debug.DrawLine(myPos, targetPos, Color.red);
                        }
                    }
                }             
    ]


2.몬스터 소환
   [ 몬스터를 반복적으로 소환하고 Destroy할 경우,데이터의 할당 자체가 어려워지고 트래픽이 삭제부분에세 계속 늘어나서 속도가 느려진다.
 
      이를 방지하기위해 몬스터를 일정량(ex.5마리)정도 한번에 소환하여 몬스터 풀로 만들어서 죽을경우 Destroy이가 아닌 SetActive를 이용하여 비활성화 시켰다가 일정시간 이 지난 이후 다시 활성화하는 식으로 진행켰다.
      
      for반복문을 사용하여 몬스터를 소환
      
      플레이어와 같이 각각의 몬스터에도 Nplayer가 부착 되어있으므로 사망시 초기화하는 함수를 사용하여, 사망후 부활 했을 때 같은 정보를 가지고 있게 초기화 시킨다.
      
      public void AfterDead(PlayerKind _kind)
       {
           Debug.Log($"<color=red>Died</color>");
           switch (_kind)
           {
               case PlayerKind.PLAYER:
                   HP = DataTblMng.Instance.GetPlayerLevelData(LEVEL).HP;
                   MP = DataTblMng.Instance.GetPlayerLevelData(LEVEL).MP;
                   break;
               case PlayerKind.MONSTER:
                   //TODO 테스트 : 몬스터 정보가져오기
                   GameManager.Instance.CheckMonsterIsServive(this);
                   //startcoroutine 순서 바꿔서 넣어주기
                   //this.gameObject.SetActive(false);
                   HP = DataTblMng.Instance.GetMonsterData(MonsterKind).HP;
                   //StartCoroutine(RespawnMon());
                   break;
           }
       }
      
      단, photon을 부착시켜야 몬스터가 사망하는것이 다른 플레이어에게도 보이므로 RPC를 쏴서 작성한다.
      RPC의 경우는 NplayerRPC라는 cs를 따로 제작하여 기능을 구현시켰다.
      [각각의 오브젝트들이 움직임을 제외한 나머지 애니메이션,UI관련 정보,각각이 가지고 있는 정보를 실시간으로 뿌려주기위해]
      *NplayerRpC는 Photon부분에서 설명
 
      보스몬스터 또한 비슷한 메커니즘으로 구현된다.
      단, 보스몬스터의 경우 플레이어가 일정한 구역에 들어갔을 때, 소환 시키는것이 맞다고 생각하여 씬이 이동하거나 보이지 않는 일정범위 콜라이더에 부딪히면 소환하게 RPC를 쏠것이고
      Target은 AllBumfferd로 구현.
      
   ]
3.UI기능 구현
   [게임 내에 보여지는 UI의 기능을 구현.
   
     UIGame이라는 프리팹을 제작하여 씬이 전환되서 게임에 들어갔을 때, 불러내어서 게임의 씬에 표시(UIGame.cs는 따로 올려 놓음)
     
     하단에 있는 요소들은 UIGame에 포함되어있는 요소들
     
     1.미니맵
       카메라를 플레이어에 부착시켜서 소환시키는 것보다, 플레이어 오브젝트에 지정범위를 설정하여 게임이 진행(게임 씬이 전환 될때, 오브젝트를 소환하는 식으로 사용: Addressable을 사용하여 오브젝트 소환을 용이하게 함)
       
       -게임내에서 구현되는데 있어서 필요한 요소중 하나이미르로 DonDestroy에 빈 오브젝트를 설치하고 그곳에 CameraManager라는 Cs를 부착후 RefreshMiniMap이라는 함수를 구현시킨다.
          public void RefreshMiniMap(Transform _transform)
          {
              if (MiniMapCamera == null)
                  return;

              MiniMapCamera.transform.localPosition=new Vector3(_transform.localPosition.x,MiniMapCamera.transform.localPosition.y,_transform.localPosition.z);
          }
       
       그리고 카메라에는 CineCam이라는 cs를 추가하여 다움과 같은 함수를 추가한다(CineCam풀 코드 따로 올려놓음)
          public void FollowTarget(Transform _target)
          {
              transform.position = _target.position;

              cameraRayTarget = CineCamera.transform.position - _target.position;
              CineCamera.transform.localPosition = new Vector3(OriPos.x, OriPos.y, OriPos.z);
              Physics.Raycast(_target.position , cameraRayTarget, out cameraRayHitInfo, Vector3.Distance(CineCamera.transform.position, _target.position));
              Debug.DrawLine(_target.position , CineCamera.transform.position, Color.magenta);

              CamaraManager.Instance.RefreshMiniMap(_target);
          }
          
        텍스쳐를 하나 추가하여 MiniMapCamera의 TargetTexture에 추가한후 CullingMask를 사용하여 보여주고자 하는 오브젝트만 보여주도록 설정한다.
       
     2.인벤
     
     3.Player의 정보(몬스터,플레이어 각각의 정보들)
       Player에 대한 정보는 Player오브젝트에 world canvas로 설정하여 오브젝트처럼 지정 위치에 표시
       -매 프레임마다 가감에 대해 표시하기 위해서 Update에 함수 대입(HP,MP)
       
       -플레이어의 HP,MP,경험치에 대한 정보는 다른 플레이어에게 보여줄 필요가 없다고 생각하여 RPC를 쏘지않았음.
        단.레벨업과 같은 특수 이벤트가 발생했을 경우에는 RPC를 발사하여 모든 플레이어들이 정보를 받아들이도록 설정
        
      -몬스터의 경우 플레이어들이 사냥을 할떄 HP에 대한 정보를 가지고 있어야하므로, HP가 가감될때마다 RPC를 쏜다.(몬스터의 사망,플레이어의 사망, 부활 포함)
      
     
   ]
  
4.게임 데이터 로드 및 세이브

5.포톤 구현
