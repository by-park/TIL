# 2021-05-11 (DVFS device tree)

### cpufreq device tree

cpu 밑에 opp 정보를 넣는다.

```
cpus {
	#address-cells = <1>;
	#size-cells = <0>;

	cpu@0 {
		compatible = "arm,cortex-a9";
		reg = <0>;
		next-level-cache = <&L2>;
		operating-points = <
			/* kHz    uV */
			792000  1100000
			396000  950000
			198000  850000
		>;
		clock-latency = <61036>; /* two CLK32 periods */
		#cooling-cells = <2>;
	};

	cpu@1 {
		compatible = "arm,cortex-a9";
		reg = <1>;
		next-level-cache = <&L2>;
	};
```



### opp list 설명 (operating points)

- cpufreq 라면 clock frequency 값들이 points가 된다.

https://www.kernel.org/doc/Documentation/devicetree/bindings/opp/opp.txt



(아래 참고) exynos document도 반영이 되는듯하다.

https://lore.kernel.org/patchwork/patch/449572/

```txt
diff --git a/Documentation/devicetree/bindings/devfreq/exynos4_bus.txt b/Documentation/devicetree/bindings/devfreq/exynos4_bus.txt
new file mode 100644
index 0000000..bd397ed
--- /dev/null
+++ b/Documentation/devicetree/bindings/devfreq/exynos4_bus.txt
@@ -0,0 +1,50 @@ 
+
+Exynos4 Memory Bus frequency driver
+-----------------------------------
+
```



### devfreq device tree

devfreq 의 exynos 관련 공식 txt 파일을 찾았다.

opp list v2는 clock frequency와 voltage 를 동시에 연동하도록 지원한다고 한다.

https://elixir.bootlin.com/linux/latest/source/Documentation/devicetree/bindings/devfreq/exynos-bus.txt

```
	bus_dmc: bus_dmc {
		compatible = "samsung,exynos-bus";
		clocks = <&cmu_dmc CLK_DIV_DMC>;
		clock-names = "bus";
		operating-points-v2 = <&bus_dmc_opp_table>;
		status = "disabled";
	};

	bus_dmc_opp_table: opp_table1 {
		compatible = "operating-points-v2";
		opp-shared;

		opp-50000000 {
			opp-hz = /bits/ 64 <50000000>;
			opp-microvolt = <800000>;
		};
		opp-100000000 {
			opp-hz = /bits/ 64 <100000000>;
			opp-microvolt = <800000>;
		};
```



### thermal zone에 dvfs cooling device 연동 방법

https://lwn.net/Articles/651273/

```
	thermal-zones {
		/* ...... */
		g3d_thermal: g3d-thermal {
			thermal-sensors = <&tmu_g3d>;
			polling-delay-passive = <0>;
			polling-delay = <0>;
			trips {
				g3d_alert_0: g3d-alert-0 {
					temperature = <30000>;	/* millicelsius */
					hysteresis = <10000>;	/* millicelsius */
					type = "active";
				};
				g3d_alert_1: g3d-alert-1 {
					temperature = <40000>;	/* millicelsius */
					hysteresis = <10000>;	/* millicelsius */
					type = "active";
				};

				/* ...... */
			};

			cooling-maps {
				map0 {
					/* Set maximum frequency as 550MHz  */
					trip = <&g3d_alert_0>;
					cooling-device = <&mali 2 2>;
				};
				map1 {
					/* Set maximum frequency as 420MHz  */
					trip = <&g3d_alert_1>;
					cooling-device = <&mali 4 4>;
				};

				/* ...... */
			};
		};

		......
	};

```



DVFS 설명

> **동적 전압 스케일링**(Dynamic Voltage Scaling, DVS)은 [컴퓨터 아키텍처](https://ko.wikipedia.org/wiki/컴퓨터_아키텍처)에서 전력 절감 기술중의 하나로서, 사용되는 부품의 인가 전압을 동적으로 높이거나 낮춤으로서 그 부품의 소비 전력을 조정한다. 동적 전압 스케일링은 에너지가 제한된 건전지를 사용하는 노트북이나 휴대용 기기에서 소비 전력을 줄이기 위해서 전압을 낮추고, 반면 컴퓨터의 성능을 높이기 위해서는 전압을 높인다. 현대의 [마이크로프로세서](https://ko.wikipedia.org/wiki/마이크로프로세서)는 동적 전압 스케일링이 [클럭 게이팅](https://ko.wikipedia.org/wiki/클럭_게이팅)과 [동적 주파수 스케일링](https://ko.wikipedia.org/wiki/동적_주파수_스케일링)과 함께 전력 절감과 성능 향상을 위해 사용 된다.

https://ko.wikipedia.org/wiki/%EB%8F%99%EC%A0%81_%EC%A0%84%EC%95%95_%EC%8A%A4%EC%BC%80%EC%9D%BC%EB%A7%81



2021.05.12 내용 추가

### cpufreq

- 커널 빌드시 config 설정 (governor 결정)

- cpufreq 초기화 함수에서 device tree 정보 파싱해서 사용
- governor 파일들
- sysfs 는 drivers/cpufreq/cpufreq.c에서 확인 가능

### devfreq

- 커널 빌드시 config 설정
- devfreq 초기화 함수에서 device tree 정보 파싱해서 사용
- governor 파일들
- sysfs는 drivers/devfreq/devfreq.c 파일에서 확인 가능

### PM QoS

- PM QoS 를 위해서 링크드 리스트로 관리

  include/linux/pm_qos.h 파일과 kernel/power/qos.c 파일 참고

  MAX 타입인지 MIN 타입인지 등 설정 (MAX 면 주어진 링크드리스트에 저장된 요청 값들 중 max 로 잡는 것이기 때문에 min 설정 값을 요청할 때 사용한다.)

https://android.googlesource.com/kernel/msm/+/refs/heads/android-msm-halibut-4.9-pie-wear-mr1/kernel/power/qos.c

```c
static BLOCKING_NOTIFIER_HEAD(cpu_dma_lat_notifier);
static struct pm_qos_constraints cpu_dma_constraints = {
	.list = PLIST_HEAD_INIT(cpu_dma_constraints.list),
	.target_value = PM_QOS_CPU_DMA_LAT_DEFAULT_VALUE,
	.target_per_cpu = { [0 ... (NR_CPUS - 1)] =
				PM_QOS_CPU_DMA_LAT_DEFAULT_VALUE },
	.default_value = PM_QOS_CPU_DMA_LAT_DEFAULT_VALUE,
	.no_constraint_value = PM_QOS_CPU_DMA_LAT_DEFAULT_VALUE,
	.type = PM_QOS_MIN,
	.notifiers = &cpu_dma_lat_notifier,
};
static struct pm_qos_object cpu_dma_pm_qos = {
	.constraints = &cpu_dma_constraints,
	.name = "cpu_dma_latency",
};
```

- pm_qos_add_notifier: pm_qos_add_request나 pm_qos_update_request 등 함수가 호출되었을 때 자동으로 동작할 함수를 등록

https://www.kernel.org/doc/Documentation/power/pm_qos_interface.txt

https://www.kernel.org/doc/html/latest/power/pm_qos_interface.html

```c
int pm_qos_add_notifier(param_class, notifier):
Adds a notification callback function to the PM QoS class. The callback is
called when the aggregated value for the PM QoS class is changed.
```

- pm_qos_add_request: pm qos 사용 전에 driver들이 각자 구조체를 만들고 이 함수로 등록을 해줘야한다. 그러면 그 후에 pm_qos_update_request 때 그 등록한 구조체를 이용해서 요청한다.

```c
void pm_qos_add_request(handle, param_class, target_value):
Will insert an element into the list for that identified PM QoS class with the
target value.  Upon change to this list the new target is recomputed and any
registered notifiers are called only if the target value is now different.
Clients of pm_qos need to save the returned handle for future use in other
pm_qos API functions.
```

- pm_qos_update_request: 내부에 pm_qos_get_value 로 PM_QOS_MIN 타입인지 PM_QOS_MAX 타입인지 정보를 얻어온다. 이 함수를 통해 pm qos 값 요청을 업데이트한다. 업데이트 요청하면 notifier 가 불리게 된다.

```c
void pm_qos_update_request(handle, new_target_value):
Will update the list element pointed to by the handle with the new target value
and recompute the new aggregated target, calling the notification tree if the
target is changed.
```

