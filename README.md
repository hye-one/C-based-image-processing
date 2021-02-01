# C-based-image-processing
C based image processing(ver.1.0)
// 영상처리 소프트웨어 Ver 0.01
#define   _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <Windows.h> // 픽셀을 출력해주는 기능 포함.
#include <math.h>
/////// 함수 선언부 (이름만 나열)
void print_menu(); 
void openImage(); 
void saveImage(); 
unsigned char** malloc2D(int, int); 
void displayImage();
void freeInputImage();  
void freeOutputImage();	
//1) 화소 점 처리 함수
void equal_image();
void brightness();	//밝게
void reverse();		//영상반전
void bw();			//흑백전환
void bwavg();		//평균값 기반 흑백처리
void gamma();		//감마변환
void parabola();	//파라볼라
void solarizing();	//솔러라이징
void poster();		//포스터라이징
void stretch();		//명암 대비 스트레치
void compress();	//명암 대비 압축
void stress();		//특정 범위 강조
void multiple();	//선명도 조절

//2) 기하학적 변환
void zoom_in();		//사진 확대
void zoom_out();	//사진 축소
void translate();	//사진 이동
void rotate();		//사진 회전
void lrMirror();	//좌우미러링
void tbMirror();	//상하미러링

/////// 전역변수 선언부
unsigned char** inImage = NULL, ** outImage = NULL;
int inH = 0, inW = 0, outH = 0, outW = 0;
char fileName[200];
// 윈도 화면용
HWND  hwnd;  HDC  hdc;

/////// 메인코드부
void main() {
	char select = 0;
	hwnd = GetForegroundWindow();   hdc = GetWindowDC(hwnd);
	while (select != '2') {
		print_menu();
		select = getche(); 
		switch (select) {
		case  '0':   openImage();   break;
		case  '1':   saveImage();   break;
		case  'A':  case 'a':  equal_image();       break;
		case  'B':  case 'b':  brightness();       break;
		case  'C':  case 'c':  reverse();       break;
		case  'D':  case 'd': bw(); break;
		case  'E':  case 'e': bwavg(); break;
		case  'F':  case 'f': lrMirror(); break;
		case  'G':  case 'g': tbMirror(); break;
		case  'H':  case 'h': gamma(); break;
		case  'I':  case 'i': parabola(); break;
		case  'J':  case 'j': solarizing(); break;
		case  'K':  case 'k': zoom_in(); break;
		case  'L':  case 'l': zoom_out(); break;
		case  'M':  case 'm': translate(); break;
		case  'N':  case 'n': rotate(); break;	
		case  'O':  case 'o': poster(); break;
		case  'P':  case 'p':stretch(); break;
		case  'Q':  case 'q':compress(); break;
		case  'R':  case 'r':stress(); break;
		case  'S':  case 's':multiple(); break;
		default: "키를 잘못 누름...";
		}
	}
	freeInputImage(); freeOutputImage();
}
///////////////////////
////// 공통 함수 정의부
void print_menu() {
	puts("                                ## 디지털 영상처리 (Beta 1) ##");
	puts("0.열기   1.저장   2.종료\n");
	puts("기본 효과...A.동일영상 B.밝게/어둡게 S:선명도 조절 C.반전 D.흑백 E.흑백(평균값 기준) R:강조\n");
	puts("특수 효과...H.감마변환 I.파라볼라 J.솔러라이징 O:포스터 P:명암대비스트레치 Q:명암대비압축\n");
	puts("기하학 변환...F.좌우미러링 G.상하미러링 K.확대 L:축소 M:이동 N:회전\n");
}
void openImage() {
	// 파일이름을 입력 받기.
	strcpy(fileName, "C:\\images\\RAW\\");
	char tmpName[100];
	printf("\n 오픈할 파일명-->");  scanf("%s", tmpName);
	strcat(fileName, tmpName);
	strcat(fileName, ".raw");
	//puts(fileName);
	// 파일을 열고, 파일의 크기를 알아내기.
	FILE* rfp;
	rfp = fopen(fileName, "rb");
	if (rfp == NULL) {
		MessageBox(hwnd, L"파일명이 없어요", L"출력창", NULL);
		return;
	}
	fseek(rfp, 0L, SEEK_END);
	long fsize = ftell(rfp); // 예 : 262144
	fclose(rfp); rfp = fopen(fileName, "rb");
	// 기존에 작업한 것이 있으면 해제
	freeInputImage();
	// 중요! 입력이미지의 높이, 폭 알아내기
	inH = inW = (int)sqrt(fsize);
	inImage = malloc2D(inH, inW);
	for (int i = 0; i < inH; i++)
		fread(inImage[i], sizeof(unsigned char), inW, rfp);
	fclose(rfp);

	equal_image();
}
unsigned char** malloc2D(int h, int w) {
	unsigned char** p;
	p = (unsigned char**)malloc(h * sizeof(unsigned char*));
	for (int i = 0; i < h; i++)
		p[i] = (unsigned char*)malloc(w * sizeof(unsigned char));
	return p;
}
void displayImage() {
	system("cls");	//화면 클리어
	unsigned char px;
	for (int i = 0; i < inH; i++)		//원본 이미지
		for (int k = 0; k < inW; k++) {
			px = inImage[i][k];
			SetPixel(hdc, k + 700, i + 300, RGB(px, px, px));
		}
	for (int i = 0; i < outH; i++) {	//변환 이미지
		for (int k = 0; k < outW; k++) {
			px = outImage[i][k];
			SetPixel(hdc, k + 50, i + 300, RGB(px, px, px));
		}
	}
}
void freeInputImage() {
	if (inImage == NULL)
		return;
	for (int i = 0; i < inH; i++)
		free(inImage[i]);
	free(inImage);
	inImage = NULL;
}
void freeOutputImage() {
	if (outImage == NULL)
		return;
	for (int i = 0; i < outH; i++)
		free(outImage[i]);
	free(outImage);
	outImage = NULL;
}
void  saveImage() {
	char sfname[200] = ("C:\\images\\RAW\\");
	char tmpFname[50];
	printf("저장할 파일명-->");
	scanf("%s", tmpFname);
	strcat(sfname, tmpFname);//
	strcat(sfname, ".raw");
	//printf("%s", sfname);
	FILE* wfp = fopen(sfname,"wb");
	for (int i = 0; i < outH; i++)
		fwrite(outImage[i], sizeof(unsigned char), outW, wfp);

	fclose(wfp);
	printf("%s으로 저장됨", sfname);
}
/////////////////////////////
////// 영상처리 함수 정의부
//1) 화소 점 처리 함수
void equal_image() {  // 동일영상 알고리즘
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	for (int i = 0; i < inH; i++)
		for (int k = 0; k < inW; k++) {
			outImage[i][k] = inImage[i][k];
		}
	displayImage();
}
void brightness() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	int value = 0;	//밝기값
	puts("밝기값을 입력하세요(정수로)\n");
	scanf("%d", &value);
	for (int i = 0; i < inH; i++)
		for (int k = 0; k < inW; k++) {
			if ((inImage[i][k] + value) > 255)
				outImage[i][k] = 255;
			else if ((inImage[i][k] + value) < 0)
				outImage[i][k] = 0;
			else
				outImage[i][k] = inImage[i][k] + value;
		}
	displayImage();
}
void bw() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	for (int i = 0; i < inH; i++)
		for (int k = 0; k < inW; k++) {
			if (inImage[i][k] < 128)
				outImage[i][k] = 0;
			else
				outImage[i][k] = 255;
		}

	displayImage();

}
void bwavg() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	int sum = 0, avg = 0;
	for (int i = 0; i < inH; i++)
		for (int k = 0; k < inW; k++) {
			sum += inImage[i][k];
			avg = sum / (outH * outW);
			if (inImage[i][k] < avg)
				outImage[i][k] = 0;
			else
				outImage[i][k] = 255;

		}
	displayImage();
}
void reverse() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	for (int i = 0; i < inH; i++)
		for (int k = 0; k < inW; k++) {
			outImage[i][k] = 255 - inImage[i][k];
		}

	displayImage();
}
void gamma() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	double gammaLCD = 2.5;
	double gamma = 1.0 / gammaLCD;
	for (int i = 0; i < inH; i++)
		for (int k = 0; k < inW; k++) {
			outImage[i][k] = 255 * pow(inImage[i][k] / 255.0, gamma);
		}
	displayImage();
}
void parabola() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	for (int i = 0; i < inH; i++)
		for (int k = 0; k < inW; k++) {
			outImage[i][k] = 255 * pow(((double)inImage[i][k] / 128.0) - 1.0, 2);
		}
	displayImage();
}
void solarizing() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	for (int i = 0; i < inH; i++)
		for (int k = 0; k < inW; k++) {
			outImage[i][k] = 255 - 255 * pow(((double)inImage[i][k] / 128.0) - 1.0, 2);
		}
	displayImage();
}
void poster() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***

	for (int i = 0; i < outH; i++) {
		for (int k = 0; k < outW; k++) {
			//명암 단계 0 32 64 96 128 160 192 224 255
			if (inImage[i][k] <= 0)
				outImage[i][k] = 0;
			else if (inImage[i][k] > 0 && inImage[i][k] <= 32)
				outImage[i][k] = 32;
			else if (inImage[i][k] > 32 && inImage[i][k] <= 64)
				outImage[i][k] = 64;
			else if (inImage[i][k] > 64 && inImage[i][k] <= 96)
				outImage[i][k] = 96;
			else if (inImage[i][k] > 96 && inImage[i][k] <= 128)
				outImage[i][k] = 128;
			else if (inImage[i][k] > 128 && inImage[i][k] <= 160)
				outImage[i][k] = 160;
			else if (inImage[i][k] > 160 && inImage[i][k] <= 192)
				outImage[i][k] = 192;
			else if (inImage[i][k] > 192 && inImage[i][k] <= 224)
				outImage[i][k] = 224;
			else
				outImage[i][k] = 255;
		}
	}
	displayImage();
}
void stretch() {	//밝기의 차이를 크게 하여 명암 대비 높임
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	int st = 30;
	for (int i = 0; i < outH; i++) {
		for (int k = 0; k < outW; k++) {
			if ((255 / (255 - 2 * st) * (inImage[i][k] - st)) > 255)
				outImage[i][k] = 255;
			else if ((255 / (255 - 2 * st) * (inImage[i][k] - st)) < 0)
				outImage[i][k] = 0;
			else
				outImage[i][k] = 255 / (255 - 2 * st) * (inImage[i][k] - st);
		}
	}
	displayImage();
}
void compress() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	double cm = 30;
	for (int i = 0; i < outH; i++) {
		for (int k = 0; k < outW; k++) {
			if ((int)((255.0 - 2.0 * cm) / 255.0 * (inImage[i][k]) + cm) > 255)
				outImage[i][k] = 255;
			else if ((int)((255.0 - 2.0 * cm) / 255.0 * (inImage[i][k]) + cm) < 0)
				outImage[i][k] = 0;
			else
				outImage[i][k] = (int)((255.0 - 2.0 * cm) / 255.0 * (inImage[i][k]) + cm);
		}
	}
	displayImage();
}
void stress() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	for (int i = 0; i < inH; i++)		
		for (int k = 0; k < inW; k++) {
			if (inImage[i][k] >= 0 && inImage[i][k] < 140)						//기본 경계값:128&192
				outImage[i][k] = inImage[i][k];
			else if (inImage[i][k] >= 180 && inImage[i][k] < 255)
				outImage[i][k] = inImage[i][k];
			else
				outImage[i][k] = 255;
		}
	displayImage();

}
void multiple() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	double c = 0;
	puts("선명도 조절(1이상 상수--->선명해짐 1이하 상수--->희미해짐");
	scanf("%lf", &c);
	for (int i = 0; i < inH; i++)
		for (int k = 0; k < inW; k++) {
			if ((int)(c * inImage[i][k]) > 255)
				outImage[i][k] = 255;
			else if ((int)(c * inImage[i][k]) < 0)
				outImage[i][k] = 0;
			else
				outImage[i][k] = (int)(c * inImage[i][k]);
		}
	displayImage();
}
//2) 기하학적 변환
void lrMirror() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	for (int i = 0; i < inH; i++)
		for (int k = 0; k < inW; k++) {
			outImage[i][k] = inImage[i][outW - 1 - k];
		}
	displayImage();
}
void tbMirror() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = inH;  outW = inW;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	for (int i = 0; i < inH; i++)
		for (int k = 0; k < inW; k++) {
			outImage[i][k] = inImage[outH - 1 - i][k];
		}
	displayImage();
}
void zoom_in() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	printf("확대비율(짝수 입력)-->");
	int scale = 0;
	scanf("%d", &scale);
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = (int)(inH*scale);  outW = (int)(inW*scale);
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	for (int i = 0; i < outH; i++)
		for (int k = 0; k < outW; k++) {
			outImage[i][k] = inImage[i/scale][k/scale];
		}
	displayImage();
}
void zoom_out(){
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	printf("축소비율(짝수 입력)-->");	
	int scale = 0;
	scanf("%d", &scale);
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	outH = (int)(inH/scale);  outW = (int)(inW/scale);
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	for (int i = 0; i < outH; i++)
		for (int k = 0; k < outW; k++) {
			outImage[i][k] = inImage[i*scale][k*scale];
		}
	displayImage();
}
void translate() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	int moveH = 0, moveW = 0;
	puts("이동할 거리를 입력하세요\n 세로방향");
	scanf("%d", &moveH);
	puts("이동할 거리를 입력하세요\n 가로방향");
	scanf("%d", &moveW);
	outH = inH+moveH;  outW = inW+moveW;
	outImage = malloc2D(outH, outW);
	double** tmpImage;
	tmpImage=malloc2D(inH, inW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	for (int i = 0; i < inH; i++)
		for (int k = 0; k < inW; k++) {
			outImage[i + moveH][k + moveW] = inImage[i][k];
		}
	displayImage();
}
void rotate() {
	if (inImage == NULL)
		return;
	// 기존에 작업한 것이 있으면 해제
	freeOutputImage();
	// 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
	
	const double PI = 3.1415926;
	int value;
	double**tmpArray=0;
	int theta=0;
	
	puts("각도를 입력하세요");
	scanf("%d", &theta);
	double rad = theta * PI / 180;
	double c=cos(rad);		double s=sin(rad);
	/*outW = (int)(inH*cos(90-rad)+inW*cos(rad)); 
	outH = (int)(inH*cos(rad)+inW*cos(90-rad));*/
	outW = inW; outH = inH;
	outImage = malloc2D(outH, outW);
	// *** 진짜 영상처리 알고리즘을 구현 ***
	int CenterH = outH / 2; // 영상의중심좌표 
	int CenterW = outW / 2; // 영상의중심좌표
	
	for (int i = 0; i < inH; i++)
		for (int k = 0; k < inW; k++) {
			// 회전변환행렬을이용하여회전하게될좌표값계산
			int newH = 0, newW = 0;
			double tmp_x = 0, tmp_y = 0;
			newH = (int)(((double)i - CenterH) * c- 
				((double)inH-(double)k - CenterW) * s + CenterH);
			newW = (int)(((double)i - CenterH) * s+ 
				((double)inH-(double)k - CenterW) * s + CenterW);
			if (newH < 0 || newH >= outH) {
				// 회전된좌표가 출력영상을위한배열값을넘어갈때
				value = 0;
			}
			else if (newW < 0 || newW >= outW) {
				// 회전된좌표가 출력영상을위한배열값을넘어갈때
				value = 0;
			}
			else {
				value = inImage[newH][newW];
			}
			outImage[i][k] = value;
		}
	displayImage();
}
