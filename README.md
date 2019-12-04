# 통신
## 라즈베리 파이 설정

### 1. 이미지와 같이 알맞게 연결
![connection](https://user-images.githubusercontent.com/48484742/70028240-78690a80-15e7-11ea-9091-35751f521c7a.JPG)

### 2. Serial 활성화 및 Bluetooth 비활성화
![2](https://user-images.githubusercontent.com/48484742/70028219-68512b00-15e7-11ea-8f5b-762480f2fd62.jpg)
![3](https://user-images.githubusercontent.com/48484742/70028306-ae0df380-15e7-11ea-9451-6141543c8d30.jpg)

* 블루투스가 uart를 사용하기 때문에 비활성화 해줘야 함
<pre>$ sudo nano /boot/config.txt</pre>
   
* 최 하단에 추가, 저장
<pre>#disable bluetooth
   dtoverlay=pi3-disable-bt</pre>
   
* bluetooth 칩간의 uart 중지
<pre>$ sudo systemctl disable hciuart</pre>
<pre>$ sudo shutdown –r now</pre>

* serial 포트 – ttyAMA0 사용 -> **Permission 문제 발생**
<pre>$ sudo chmod 777 /dev/ttyAMA0</pre>
   
* 포트 Baudrate 확인
<pre>$ stty -F /dev/ttyAMA0</pre>
   
* 포트 Baudrate 변경
<pre>$ stty -F /dev/ttyAMA0 115200</pre>

### 3. putty를 이용한 통신 상태 확인     
#### 3-1. 라즈베리파이
* 라즈베리파이 putty 설치
<pre>$ sudo apt-get install putty</pre>
<pre>$ sudo putty</pre>
* 포트와 baudrate 알맞게 입력 후 이미지와 같이 putty open

![5](https://user-images.githubusercontent.com/48484742/70028605-6e93d700-15e8-11ea-9f40-3f991b81cf95.jpg)

#### 3-2. 윈도우
* [putty 설치](https://putty.ko.softonic.com/)
* [PL2303 drivers 설치](http://www.prolific.com.tw/US/ShowProduct.aspx?p_id=225&pcid=41)
* 장치관리자에서 포트 번호 및 baudrate 확인 및 설정

![6](https://user-images.githubusercontent.com/48484742/70028477-1eb51000-15e8-11ea-9469-d300176c283d.jpg)
* 이미지와 같이 putty 실행(window)

![7](https://user-images.githubusercontent.com/48484742/70028590-6045bb00-15e8-11ea-9610-c8f9a2abd3f4.jpg)

#### 3-3 라즈베리파이 IP, PW 입력 후 연결 확인
![8](https://user-images.githubusercontent.com/48484742/70028748-d813e580-15e8-11ea-901d-aefff7b7d7cc.jpg)

참고:
* https://cccding.tistory.com/93

### 4. 키보드 입력 받아오기
* wiringpi 라이브러리 사용

#### 4-1. 라즈베리파이
* wiringpi 설치 방법 1
<pre> $ sudo apt-get install wiringpi</pre>

* wiringpi 설치 방법 2
<pre>
$ sudo apt-get install git-core
$ sudo apt-get update
$ sudo apt-get upgrade
$ cd
$ git clone git://git.drogon.net/wiringPi
</pre>

* -> 191104기준 네트워크 접근 에러
* -> 조사결과 서버를 닫았거나 너무 느린상태
<pre>$ git clone https://github.com/wiringpi/wiringpi</pre>

참고: https://github.com/WiringPi/WiringPi-Python/issues/40

<pre>
$ cd ~/wiringPi
$ git pull origin
$ cd ~/wiringPi
$ ./build - wiringPi 빌드
</pre>

참고: 
* http://wiringpi.com/download-and-install/
* https://feelsoverygood.tistory.com/34

* uart_test.c 작성
```
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <wiringPi.h>
#include <wiringSerial.h>

int main ()
{
  int fd;
  int data;

  if ((fd = serialOpen ("/dev/ttyAMA0", 115200)) < 0)
  {
    fprintf (stderr, "Unable to open serial device: %s\n", strerror (errno));
    return 1;
  }

  printf ("\nRaspberry Pi UART Test");

  while(1)
  {
    data = serialGetchar(fd);
    printf ("\nPC > RPi = %c", (char)data);
    serialPutchar(fd, data);
    serialPuts(fd, "\n");
    fflush(stdout);
  }

  return 0 ;
}
```

* 컴파일
<pre> $ gcc uart_test.c -o uart_test –lwiringPi</pre>
프로그램 실행
<pre> $ sudo ./uart_test </pre>

![9](https://user-images.githubusercontent.com/48484742/70028690-b0248200-15e8-11ea-8811-a499d5fe9165.jpg)

참고:https://hongci.tistory.com/8
http://wiringpi.com/reference/serial-library/

### 5. pc 통신(기본 연습)

* Windows.h 사용
* (윈도우 개발자들이 필요한 모든 매크로들, 다양한 함수들과 서브시스템에서 사용되는 모든 데이터 타입들 그리고 윈도우 API의 함수들을 위한 정의를 포함하는 윈도우의 C 및 C++ 헤더 파일이다.)

```
#include<stdio.h>
#include<Windows.h>
```

* **Opening the serial port**
* 3-2와 같이 serial port 확인 후 알맞게 입력

```
HANDLE hSerial;

hSerial = CreateFile("\\\\.\\COM4", GENERIC_READ | GENERIC_WRITE, 0, 0, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, 0);

if (hSerial == INVALID_HANDLE_VALUE) {
  if (GetLastError() == ERROR_FILE_NOT_FOUND) {
    printf("Serial port not found!\n");
  }
}
else {
  printf("Serial port found!\n");
}
```

* **Setting Parameter**
* DCB 구조체 사용

```
DCB dcbSerialParams = { 0 };

dcbSerialParams.DCBlength = sizeof(dcbSerialParams);
	
//COM 시리얼 포트의 상태를 체크
if (!GetCommState(hSerial, &dcbSerialParams)) {
  // Error getting COM port state
  printf("Error : Getting COM port state\n");
}
// 시리얼 통신 설정
dcbSerialParams.BaudRate = CBR_115200;      // 보드레이트 설정
dcbSerialParams.ByteSize = 8;             // 데이터 사이즈 설정
dcbSerialParams.StopBits = ONESTOPBIT;    // 스탑비트 설정
dcbSerialParams.Parity = NOPARITY;      // 패리티 설정

if (!SetCommState(hSerial, &dcbSerialParams)) {     // 설정 적용
// Error setting COM port state
  printf("Error : Setting COM port state\n");
}
```

* **Setting timeouts**
* serial port 통신의 문제 -> 통신시 데이터가 없는 경우 읽기를 시도하면 프로그램이 중단될 수 있음 
* 해결방법 (1) thread 사용 (2) COMMTIMEOUTS 구조체 사용, 타임아웃 설정

```
COMMTIMEOUTS timeouts = { 0 };
timeouts.ReadIntervalTimeout = 50;// 타이밍 아웃 전 수신 문자 사이에서의 대기시간(ms)
timeouts.ReadTotalTimeoutConstant = 50;// 반환하기 전 대기시간(ms)
timeouts.ReadTotalTimeoutMultiplier = 10;// 읽기 작업에서 요청된 데이터 반환 전 대기 시간(ms)

timeouts.WriteTotalTimeoutConstant = 50;
timeouts.WriteTotalTimeoutMultiplier = 10;

if (!SetCommTimeouts(hSerial, &timeouts)) {
		// Error set timeouts
		printf("Error : Setting timeouts\n");
	}
```

참고:
* Bayer, R. (2008). Windows serial port programming. línea][consultado 13/11/2012]< http://www. robbayer. com/serial. php.
* https://blog.naver.com/leesoo9297/221162382305
#### 5-1. Send
* WriteFile(Handle, 데이터 버퍼, 쓸 바이트 수,실제로 쓰는 바이트 수로 설정할 정수의 포인터)
```
DWORD dwBytesWrite = 0;
	int writeSize = 10;
	unsigned char str[11] = "S012345678"; //writeSize + 1 크기로 해야됨 -> 공백 안생김(공백은 stop bit)
	while (1) {
		if (WriteFile(hSerial, str, writeSize, &dwBytesWrite, NULL)) {
			// Write Complete
		}
	}
```

#### 5-2. Recieve
* readSize는 writeSize과 같다
* ReadFile(Handle, 데이터 저장 버퍼, 읽을 바이트 수,실제로 읽은 바이트 수로 설정할 정수의 포인터)
```
DWORD dwBytesRead = 0;
	int readSize = 10;
	unsigned char buff[10];
	unsigned char T_buff[10];
	int start = 0, index = 0;;
	
	if (ReadFile(hSerial, buff, readSize, &dwBytesRead, NULL)) {
		for (int i = 0; i < 10; i++) {
			printf("%c ", buff[i]);    // Print data sample
		}
		printf("\n");
	}
	//시작점 잡기
	for (int i = 0; i < 10; i++) {
		if (buff[i] == 'S') {
			start = i;
			break;
		}
	}
	//시작점 기준으로 원본 데이터 찾기
	if (start != 9) {
		for (int i = start+1; i < 10; i++) {
			T_buff[index++] = buff[i];
		}
	}
	for (int i = 0; i < start+1; i++) {
		T_buff[index++] = buff[i];
	}
	for (int i = 0; i < 10; i++) {
		printf("%c ", T_buff[i]);    // Print data sample
	}
	printf("\n"); 
```

#### 5-3. Closing
```
CloseHandle(hSerial);
```
#### 결과
![10](https://user-images.githubusercontent.com/48484742/70048260-3357cf00-160d-11ea-872e-e88790ef5556.jpg)
