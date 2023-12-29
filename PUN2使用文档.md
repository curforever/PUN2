# PUN2使用文档



## 1 PUN2快速入门配置

### 1.1 客户端A连接到PUN2服务器

> [Photon文档](https://doc.photonengine.com/zh-cn/fusion/current/getting-started/fusion-intro)

1. [Photon官网](https://www.photonengine.com/pun)注册账号并CreateApp（AppID后面要用）

2. UnityAssetStore找到**PUN2-FREE**并Import

3. 在弹窗或Window/PUN/HighlightServerSettings里设置**AppID**

   <img src="PUN2使用文档.assets/image-20231228204418842.png" alt="image-20231228204418842" style="zoom: 50%;" />

   <img src="PUN2使用文档.assets/image-20231228204921313.png" alt="image-20231228204921313" style="zoom: 80%;" />

4. Window/PUN/HighlightServerSettings的一些其他设置

   - AppVersion: 1
   - Fixed Region：asia
   - NetworkLogging：ALL
   - PUNLogging: Full

5. Hierarchy窗口下新建空节点挂载Launcher.cs

   ```cs
   using System.Collections;
   using System.Collections.Generic;
   using UnityEngine;
   using Photon.Pun;
   
   public class Launcher : MonoBehaviourPunCallbacks
   {
       void Start()
       {
           PhotonNetwork.ConnectUsingSettings();
       }
   
       public override void OnConnectedToMaster()
       {
           base.OnConnectedToMaster();
           Debug.Log("Welcome SEU");
       }
   
       void Update()
       {
   
       }
   }
   ```

   运行后观察Console窗口，发现输出了想要的Welcome SEU，说明成功连接到服务器

   ![image-20231228205436819](PUN2使用文档.assets/image-20231228205436819.png)



### 1.2 对应客户端的人物生成和控制

1. 房间创建+NickName，更新Launcher.cs

   ```cs
   using System.Collections;
   using System.Collections.Generic;
   using UnityEngine;
   using UnityEngine.UI;
   using Photon.Pun;
   using Photon.Realtime;
   
   public class Launcher : MonoBehaviourPunCallbacks
   {
       public string RoomName = "Room";
       private string gameVersion = "1";
   
       [SerializeField]
       private const byte maxPlayersPerRoom = 4;
   
       void Awake()
       {
           PhotonNetwork.AutomaticallySyncScene = true;	// 确保该变量为 true，否则无法同步
       }
   
       void Start()
       {
           if (PhotonNetwork.IsConnected)  // 已连接，加入到房间
           {
               PhotonNetwork.JoinOrCreateRoom(RoomName, new RoomOptions() { MaxPlayers = maxPlayersPerRoom }, default);
           }
           else    // 未连接  
           {
               PhotonNetwork.ConnectUsingSettings();		
               PhotonNetwork.GameVersion = gameVersion;
           }
       }
   
       public override void OnConnectedToMaster()	// 运行ConnectUsingSettings()后会先连接到 Master节点，再创建或加载房间
       {
           Debug.Log("Launcher.cs: OnConnectedToMaster() was called.");
           PhotonNetwork.JoinOrCreateRoom(RoomName, new RoomOptions() { MaxPlayers = maxPlayersPerRoom }, default);
       }
   
       public override void OnJoinedRoom()
       {
           // 加入房间后用NickName进行区分
           if (PhotonNetwork.PlayerList.Length == 1)
           {
               PhotonNetwork.LocalPlayer.NickName = "Player1";
           }
           else if (PhotonNetwork.PlayerList.Length == 2)
           {
               PhotonNetwork.LocalPlayer.NickName = "Player2";
           }
   
           Debug.Log("Launcher.cs: Joined Room Successfully!");
           Debug.Log("Launcher.cs: You Are " + PhotonNetwork.LocalPlayer.NickName);
       }
   }
   ```

   ![image-20231228212447391](PUN2使用文档.assets/image-20231228212447391.png)

2. Player创建、移动脚本、同步组件

   - 创建Player的Prefab

   - 添加rigidBody组件（gravity重力）

   - 添加PlayerControl.cs控制移动（以下脚本不依赖相机视角控制方向）

     ```cs
     using System.Collections;
     using System.Collections.Generic;
     using UnityEngine;
     using Photon.Pun;
     using Photon.Realtime;
     
     public class PlayerControl : MonoBehaviourPunCallbacks
     {
         void Update()
         {
             // 确保各自控制各自
             if(!photonView.IsMine && PhotonNetwork.IsConnected)
                 return;
             float vertical = Input.GetAxis("Vertical");
             float horizontal = Input.GetAxis("Horizontal");
             Vector3 dir = new Vector3(horizontal, 0, vertical);
     
             if (dir != Vector3.zero) {
                 transform.rotation = Quaternion.LookRotation(dir);
                 transform.Translate(Vector3.forward * 2 * Time.deltaTime);
             } 
         }
     }
     ```

   - 添加**Photon View**组件和**Photon Transform View**组件

3. 创建好Player1和Player2的Prefab（目前为红蓝区分）

   - 把两个Player作为Prefab拖拽到Project-Assets/Photon/PhontonUnityNetworking/Resources中

   - 代码中加入Instantiate实例化的逻辑，更新Launcher.cs

     ```cs
     using System.Collections;
     using System.Collections.Generic;
     using UnityEngine;
     using UnityEngine.UI;
     using Photon.Pun;
     using Photon.Realtime;
     
     public class Launcher : MonoBehaviourPunCallbacks
     {
         public string RoomName = "Room";
         private string gameVersion = "1";
         [SerializeField]
         private const byte maxPlayersPerRoom = 4;
     
         void Awake()
         {
             PhotonNetwork.AutomaticallySyncScene = true;	// 确保该变量为 true，否则无法同步
         }
     
         void Start()
         {
             if (PhotonNetwork.IsConnected)  // 已连接，加入到房间
             {
                 PhotonNetwork.JoinOrCreateRoom(RoomName, new RoomOptions() { MaxPlayers = maxPlayersPerRoom }, default);
             }
             else    // 未连接  
             {
                 PhotonNetwork.ConnectUsingSettings();		
                 PhotonNetwork.GameVersion = gameVersion;
             }
             
         }
     
         public override void OnConnectedToMaster()	// 运行ConnectUsingSettings()后会先连接到 Master节点，再创建或加载房间
         {
             Debug.Log("Launcher.cs: OnConnectedToMaster() was called.");
             PhotonNetwork.JoinOrCreateRoom(RoomName, new RoomOptions() { MaxPlayers = maxPlayersPerRoom }, default);
         }
     
         public override void OnJoinedRoom()
         {
             // 加入房间后用NickName进行区分
             if (PhotonNetwork.PlayerList.Length == 1)
             {
                 PhotonNetwork.LocalPlayer.NickName = "Player1";
                 PhotonNetwork.Instantiate("RedPlayer", new Vector3(-21,2,-7), Quaternion.identity, 0);
             }
             else if (PhotonNetwork.PlayerList.Length == 2)
             {
                 PhotonNetwork.LocalPlayer.NickName = "Player2";
                 PhotonNetwork.Instantiate("BluePlayer", new Vector3(-21,2,-7), Quaternion.identity, 0);
             }
     
             Debug.Log("Launcher.cs: Joined Room Successfully!");
             Debug.Log("Launcher.cs: You Are " + PhotonNetwork.LocalPlayer.NickName);
         }
     }
     ```

     

> 下一步：
>
> - 摄像机第一人称的挂载
> - 鼠标控制方向键盘控制移动
> - Nickname逻辑优化
> - 人物入场时可选择，而不是固定
> - ...
>
> 上一版本文档遗留的记录（防止后面用到所以保留了下来）：
>
> - 改为Manual，拖拽Transition下来。同步改为manual，不然只能看见对面的人，没法同步动作
