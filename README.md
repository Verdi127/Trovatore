## nRF52840 + OLED 2개 연결 예시

OLED는 I2C 장치라서 `SCK`가 아니라 `SCL`을 사용합니다. ZMK에서는 두 개의 0.91" OLED를 각각 다른 I2C 버스에 연결하면 가장 단순하게 구성할 수 있습니다.

아래 예시는 다음 배선을 기준으로 합니다.

- OLED 1: SDA = 17, SCL = 20
- OLED 2: SDA = 22, SCL = 24

두 버스로 분리하면 두 OLED가 같은 주소 `0x3C`를 써도 됩니다.

```dts
&pinctrl {
	i2c0_default: i2c0_default {
		group1 {
			psels = <NRF_PSEL(TWIM_SDA, 0, 17)>,
					<NRF_PSEL(TWIM_SCL, 0, 20)>;
		};
	};

	i2c1_default: i2c1_default {
		group1 {
			psels = <NRF_PSEL(TWIM_SDA, 0, 22)>,
					<NRF_PSEL(TWIM_SCL, 0, 24)>;
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

원하면 다음 단계로는 보드 이름에 맞춘 `.overlay` 파일 형태로 바로 정리해드릴 수 있습니다.

## GitHub Actions 자동 펌웨어 빌드

이 저장소는 푸시 시 자동으로 ZMK 펌웨어를 빌드하도록 설정되어 있습니다.

- 워크플로 파일: `.github/workflows/build.yml`
- 빌드 매트릭스: `build.yaml`
- West 매니페스트: `config/west.yml`

현재 `build.yaml`은 CI 동작 확인을 위해 `settings_reset` 타깃으로 되어 있습니다.
실제 펌웨어로 바꾸려면 `build.yaml`에서 `board`/`shield`를 본인 하드웨어에 맞게 수정하세요.

예시:

```yaml
include:
	- board: nice_nano_v2
		shield: your_shield_left
		artifact-name: your-fw-left
	- board: nice_nano_v2
		shield: your_shield_right
		artifact-name: your-fw-right
```

빌드 결과물은 GitHub Actions 실행 상세 화면의 Artifacts에서 다운로드할 수 있습니다.
