## nRF52840 + OLED 2개 연결 예시

OLED는 I2C 장치라서 `SCK`가 아니라 `SCL`을 사용합니다. ZMK에서는 두 개의 0.91" OLED를 각각 다른 I2C 버스에 연결하면 가장 단순하게 구성할 수 있습니다.

아래 예시는 다음 배선을 기준으로 합니다.

- OLED 1: SDA = 20, SCL = 17
- OLED 2: SDA = 24, SCL = 22

두 버스로 분리하면 두 OLED가 같은 주소 `0x3C`를 써도 됩니다.

```dts
&pinctrl {
	i2c0_default: i2c0_default {
		group1 {
	 		psels = <NRF_PSEL(TWIM_SDA, 0, 20)>,
	 					<NRF_PSEL(TWIM_SCL, 0, 17)>;
		};
	};

	i2c1_default: i2c1_default {
		group1 {
	 		psels = <NRF_PSEL(TWIM_SDA, 0, 24)>,
	 					<NRF_PSEL(TWIM_SCL, 0, 22)>;
		};
	};
};

&i2c0 {
	status = "okay";
	pinctrl-0 = <&i2c0_default>;
	pinctrl-names = "default";

	oled0: ssd1306@3c {
		compatible = "solomon,ssd1306fb";
		reg = <0x3c>;
		width = <128>;
		height = <32>;
	};
};

&i2c1 {
	status = "okay";
	pinctrl-0 = <&i2c1_default>;
	pinctrl-names = "default";

	oled1: ssd1306@3c {
		compatible = "solomon,ssd1306fb";
		reg = <0x3c>;
		width = <128>;
		height = <32>;
	};
};
```

같은 I2C 버스에 두 OLED를 연결하고 싶다면, 한쪽 주소를 `0x3D`로 바꿔야 합니다. 두 화면을 동시에 같은 내용으로 띄우려면 ZMK 쪽에서 추가 설정이 필요할 수 있습니다.

위 배선은 실제 파일로 다음 위치에 반영되어 있습니다.

- `config/corne_left.overlay`
- `config/corne_right.overlay`
- `config/corne.conf`

## GitHub Actions 자동 펌웨어 빌드

이 저장소는 푸시 시 자동으로 ZMK 펌웨어를 빌드하도록 설정되어 있습니다.

- 워크플로 파일: `.github/workflows/build.yml`
- 빌드 매트릭스: `build.yaml`
- West 매니페스트: `config/west.yml`
- ZMK 패치 파일: `patches/zmk-dual-oled.patch`

현재 `build.yaml`은 `nice_nano_v2 + corne_left/right` 타깃으로 설정되어 있습니다.

### 듀얼 OLED 서로 다른 화면

포크 없이도 동작하도록, GitHub Actions 빌드 시 ZMK 원본 소스에 `patches/zmk-dual-oled.patch`를 자동 적용합니다.

- OLED 1(`oled0`): 기본 상태 화면
- OLED 2(`oled1`): 별도 상태 화면(배터리/레이어 + `OLED 2` 라벨)

주의:

- 이 방식은 ZMK upstream 변경으로 패치가 깨질 수 있습니다.
- 패치 적용 실패 시 Actions 로그의 `Apply dual OLED patch` 단계에서 확인할 수 있습니다.

예시:

```yaml
include:
  - board: nice_nano_v2
    shield: corne_left
    artifact-name: trovatore-left
  - board: nice_nano_v2
    shield: corne_right
    artifact-name: trovatore-right
```

빌드 결과물은 GitHub Actions 실행 상세 화면의 Artifacts에서 다운로드할 수 있습니다.
