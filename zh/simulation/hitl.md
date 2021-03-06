# 硬件在环仿真（HITL）

硬件在环仿真模式 (HITL 或 HIL) 下 PX4 固件代码运行在真实的飞行控制器硬件平台上。 这种方法的优点是可以在实际硬件上测试大多数的实际飞行代码。

HITL 模式下 PX4 支持多旋翼 (使用 jMAVSim 或者 Gazebo) 和固定翼 (使用 Gazebo 或者 X-Plane demo/full version) 无人机的仿真。

<a id="compatible_airframe"></a>

## HITL兼容机架

目前兼容的机架构型和模拟器的情况如下：

| 机架                                                                                                      | `SYS_AUTOSTART` | Gazebo | jMAVSim |
| ------------------------------------------------------------------------------------------------------- | --------------- | ------ | ------- |
| <a href="../airframes/airframe_reference.md#copter_simulation_(copter)_hil_quadcopter_x">HIL Quadcopter X</a>                                                                               | 1001            | Y      | Y       |
| <a href="../airframes/airframe_reference.md#vtol_standard_vtol_hil_standard_vtol_quadplane">HIL Standard VTOL QuadPlane</a>                                                                               | 1002            | Y      |         |
| [Generic Quadrotor x](../airframes/airframe_reference.md#copter_quadrotor_x_generic_quadrotor_x) copter | 4001            | Y      | Y       |
| [DJI Flame Wheel f450](../airframes/airframe_reference.md#copter_quadrotor_x_dji_flame_wheel_f450)      | 4011            | Y      | Y       |

<a id="simulation_environment"></a>

## HITL 仿真环境

硬件在环仿真（HITL）模式下标准的 PX4 固件在真实的硬件上运行。 JMAVSim 或 Gazebo (运行在开发计算机上) 通过 USB/UART 完成与飞行控制器硬件平台的连接。 模拟器充当在 PX4 和 *QGroundControl* 之间共享 MAVLink 数据的网关。

> **Note** 如果飞行控制器支持网络连接且使用的是稳定、低延迟的连接（如有线以太网，WIFI 通常不太稳定），那么模拟器也可以使用 UDP 完成通讯连接。 例如，该配置已经使用一台运行 PX4 且通过以太网连接到开发计算机的 Raspberry Pi 进行了验证测试 (包括 jMAVSim 运行命令的启动配置在 [这里](https://github.com/PX4/Firmware/blob/master/posix-configs/rpi/px4_hil.config))。

下图展示了仿真模拟的环境：
* 飞控板 HITL 模式被激活 (通过 *QGroundControl*) ，该模式下不会启动飞控板上任何传感器。
* *jMAVSim* 或者 *Gazebo* 通过 USB 连接到飞控板。
* 模拟器通过 UDP 连接到 *QGroundControl* 并将 MAVLink 数据传输至 PX4 。
* (可选) 通过串口可将操纵杆/游戏手柄通过 *QGroundControl* 连接至仿真回路中。
* (可选 - 仅适用于Gazebo) Gazebo 还可以连接到一个 offboard API ，并将 MAVLink 数据桥接到 PX4 。

![HITL 配置 - jMAVSim 和 Gazebo](../../assets/simulation/px4_hitl_overview_jmavsim_gazebo.png)


## HITL 相比于 SITL

SITL 开发计算机中的模拟环境中运行, 并使用专门为该环境生成的固件。 除了仿真程序从模拟器中获取虚假的环境数据外，系统的行为也很正常。

相比之下， HITL 在正常飞控硬件平台上运行正常的处于 ”HITL 模式“ 的 PX4 固件。 仿真数据进入整个仿真系统的时间点与 SITL 有所不同。 指令器和传感器等有 HIL 模式的核心模块在启动时被绕过了一些正常的功能。

总而言之， HITL 在真实硬件上运行标准 PX4 固件，而 SITL 实际上要比标准 PX4 系统执行更多的代码。


## 配置 HITL

### JMAVSim/Gazebo HITL 仿真环境

1. 通过 USB 将自动驾驶仪直接连接到 *QGroundControl*。
1. 激活 HITL 模式
   1. 打开 **Setup > Safety** 选项卡。
   1. 在 *HITL Enabled* 下拉框中选择 **Enabled** 完成 HITL 模式的激活。

      ![QGroundControl HITL 配置](../../assets/gcs/qgc_hitl_config.png)
1. 选择机架
   1. 打开 **Setup > Airframes** 选项卡。
   1. 选择一个你想要进行测试的 [兼容的机架](#compatible_airframe) 。 Then click **Apply and Restart** on top-right of the *Airframe Setup* page.

      ![选择机架](../../assets/gcs/qgc_hil_config.png)
1. 如有必要, 校准您的 RC 遥控器 或操纵杆。
1. 设置 UDP
   1. 在设置菜单的 "*General*" 选项卡下, 取消选中 *AutoConnect* 一栏中除 **UDP** 外的所有复选框。

      ![GITL 模式 QGC 自动连接设置](../../assets/gcs/qgc_hitl_autoconnect.png)
1. (可选) 配置操纵杆和故障保护。 设置以下 [parameters](https://docs.px4.io/en/advanced_config/parameters.html#finding-a-parameter) 以便使用操纵杆而不是 RC 遥控器：
   * [COM_RC_IN_MODE](../advanced/parameter_reference.md#COM_RC_IN_MODE) 更改为 "Joystick/No RC Checks". 这允许操纵杆输入并禁用 RC 输入检查。 这允许操纵杆输入并禁用 RC 输入检查。
   * [NAV_DLL_ACT](../advanced/parameter_reference.md#NAV_DLL_ACT) 更改为 "Disabled"。 这可确保在没有无线遥控的情况下运行 HITL 时 RC 失控保护不会介入。

   > **Tip** *QGroundControl User Guide* 中也有如何配置 [操纵杆](https://docs.qgroundcontrol.com/en/SetupView/Joystick.html) 和 [虚拟操纵杆](https://docs.qgroundcontrol.com/en/SettingsView/VirtualJoystick.html) 的说明。

完成所有的配置设定后 **关闭** *QGroundControl* 并断开飞控板与计算机的连接。

### X-Plane HITL 仿真环境

遵循以下流程进行 X-Plane 模拟器的配置：

#### Gazebo

> **Note** 确保 *QGroundControl* 没有运行！

1. 更新环境变量：
   ```sh
   cd <Firmware_clone>
    make px4_sitl_default gazebo
   ```
1. 打开载具模型的 sdf 文件（例如 **Tools/sitl_gazebo/models/iris/iris.sdf**）。
1. 找到文件的 `mavlink_interface plugin` 分区，将 `serialEnabled` 和 `hil_mode` 参数更改为 `true` 。

   ![HIL 参数](../../assets/simulation/gazebo_sdf_model_hil_params.png)

   > **Note**  iris.sdf文件时自动生成的。 因此，您需要保存您修改过的文件的副本，或者为每个版本重新编辑。
1. 如有必要的话替换掉 `serialDevice` 参数 (`/dev/ttyACM0`) 。

   > **Note** 串口设备参数取决于载具与计算机使用哪个端口完成连接 (通常情况下都是 `/dev/ttyACM0`)。 在 Ubuntu 上最简单的一个检测办法就是将自驾仪插入电脑，然后打开终端窗口输入 `dmesg | grep "tty"` 命令。 命令执行结果中最后一个显示的设备就是我们关心的。

1. 设置环境变量
   ```sh
   source Tools/setup_gazebo.bash $(pwd) $(pwd)/build/px4_sitl_default
   ```
   在 HITL 模式下运行 Gazebo ： sh gazebo Tools/sitl_gazebo/worlds/iris.world
   ```sh
   gazebo Tools/sitl_gazebo/worlds/iris.world
   ```
1. 开启 *QGroundControl*。 它应该会自动连接 PX4 和 Gazebo 。

<a id="jmavsim_hitl_configuration"></a>

#### jMAVSim (仅适用于四旋翼无人机)

> **Tip** Gazebo 还支持与offboard API 共享 MAVLink 数据！

1. 将飞行控制器连接到计算机, 并等待其启动。
1. 在 HITL 模式下运行 jMAVSim (r如有必要，修改串口号名称 `/dev/ttyACM0` - 比如，在 Mac OS 上该参数应为 `/dev/tty.usbmodem1`)： sh ./Tools/jmavsim_run.sh -q -d /dev/ttyACM0 -b 921600 -r 250
   ```sh
   ./Tools/jmavsim_run.sh -q -s -d /dev/ttyACM0 -b 921600 -r 250
   ```

   > **Note** Replace the serial port name `/dev/ttyACM0` as appropriate. 在 macOS 上，这个端口将是 `/dev/tty.bankmodem1`。 在 Windows 上(包括Cygwin) 它将是 COM1 或另一个端口 - 请检查 Windows 设备管理器中的连接。
1. 开启 *QGroundControl*。 它应该会自动连接 PX4 和 Gazebo 。


## 在 HITL 仿真中执行自主飞行任务

总而言之， HITL 在真实硬件上运行标准 PX4 固件，而 SITL 实际上要比标准 PX4 系统执行更多的代码。
