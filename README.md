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

3.UI기능 구현

4.게임 데이터 로드 및 세이브

5.포톤 구현
