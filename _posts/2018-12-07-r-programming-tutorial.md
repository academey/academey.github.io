---
layout: post
title: "빅 데이터 분석과 R 프로그래밍(기초Ⅰ) (POSTECHX)"
author: academey
categories: machine-learning
---

# 1. R 기본 스크립트와 설명

R은 오픈소스 통계분석 프로그램

1. rnorm(Random Sampling from normal distribution)
   rnorm 은 랜덤 정규 분포 값 생성해줌. (디폴트 평균은 0, 표준편차 1) rnorm(n, mean, std)
   x3 <- rnorm(100)
   head(x3) # x3 벡터의 앞 6개 원소만 보여줌

2. 패키지 설치
   install.packages("ggplot2")
   library(ggplot2)
   help(ggplot2)
   help.search("support vector")

3. seq func
   z <- seq(-10, 10, 0.01)
   -10 부터 10까지 0.01 단위로 짤라 벡터리스트를 만들어준다.

4. 벡터에서 값 삭제
   x<-c(1,3,5,7,9)
   x[-1] # 첫번 째 값 삭제
   x[-(1,2)] # 첫, 두 번째 값 삭제
   x[-(1:3)] # 1~3 번째 값 삭제

5. Seq 함수로 벡터 생성
   Y1<-seq(0,10, length=20) # 0부터 10까지 20개의 벡터 생성
   Y1<-seq(0,10, by=20) # 0부터 10까지 0.5 간격으로 벡터 생성

6. Rep 함수로 벡터 생성 (Replication)
   Z1<-rep(1:4, 2) # 1부터 4까지 2번 반복해서 생성
   Z2<-rep(1:2,5) # 1부터 2까지 5번 반복해서 생성

7. 벡터 결합 (cbind , 컬럼 기준으로 결합 / rbind, 로우 기준으로 결합)
   x<-c(1,3,5,7,9)
   C1<-c(2,4,6,8,10)
   C2<-cbind(x, c1) # C2 는 (5,2) 행렬이 된다. 1열에 1,2 2열에 3,4 ….
   C3<-rbind(x,c1) #C3은 (2,5) 행렬이 된다. 1열에 1,3,5,7,9 2열에 2,4,6,8,10

8. Matrix 함수로 행렬 생성
   M1<-matrix(1:10, nrow=2) # 1부터 10까지, row 개수는 2개
   M2<-matrix(1:6, ncol=3) # 1부터 6까지, column 개수가 3개로 생성된다.
   1열 부터 채우는 게 디폴트 인데, byrow=T를 주면 1행부터 채워줌.

고차원 행렬을 주려면, array를 이용해서 생성한다.
A1<-array(c(1:18), dim=c(3,3,2)) # 1부터 18까지의 값을 3X3 행렬 2개에 넣어준다.
열부터 채워지면서
1 4 7
2 5 8
3 6 9
10 13 16
11 14 17
12 151 8 이런 형태가 된다.

9. 객체 이름 정의
   gender<-c(0,1)
   names(gender)<-c("female", "male")
   각 벡터에 이름을 부여해준다.

10. 범주형 변수 생성 (factor 사용)
    size<-c("S", "M","L","XL")

# define size as a factor (categorical variable)

size_factor<-factor(size) # 팩터에는 순서가 없다.
size_factor2 <- factor(size,
levels = c("S", "M","L","XL”)) # 이렇게 입력 받을 수도 있다.
Is.factor(size_factor) # 요걸로 팩터인지 아닌지 확인할 수 있다.

순서 있게 만들려면, 다음과 같이 ordered property 를 넣어준다.
size_factor3 <- factor(size, ordered = TRUE,
levels = c("S", "M","L","XL"))

11. 행렬에 이름 넣어주기 (dimnames, colnames, row names)
    x<-matrix(rnorm(12),nrow=4)

# check dimension of x

dim(x)

이제 디멘젼(row 또는 column)에 이름을 넣어보자.
[[1]]은 행을 의미하고 [[2]]는 열을 의미한다.
x 라는 기호에 1부터 3까지 더해서 붙여준다.. 그러면 x1, x2, x3 을 준다.
dimnames(x)[[2]]<-paste("x",1:3,sep=“”) # 이렇게 하면 열의 이름에 x1, x2, x3 이 써져있다.

아래와 같이 하면 행에 id1,id2,id3,id4 를 붙여준다. 두개는 같은 결과다.
dimnames(x)[[1]]<-paste(“id”,1:4,sep:””)
rownames(x) <- c(“id1”,”id2”,”id3”,”id4”)

12. 행렬과 데이터 프레임
    이렇게 만든 행렬들이, 데이터 프레임인가? (데이터 프레임이 뭔지는 아직 모르겠음)
    데이터 프레임은 객체값을 행렬로 저장할 뿐 아니라 변수명, 관측치번호 등 여러가지 정보를 가지는 객체. 따라서 행렬을 데이터 프레임으로 인식하기 위해서는 as.data.frame()으로 정의 필요.
    그걸 확인하고, 행렬을 데이터 프레임으로 지정해보자.

# data frame

is.data.frame(x)

# matrix x is not data frame

# define x as a data frame

x<-as.data.frame(x)

# then x is a data frame

is.data.frame(x)

이렇게 x 가 데이터 프레임이 되면, 각 열은 변수열, 각 행은 관측치가 된다.
특정 변수의 평균 및 편차들을 계산할 수 있다.

x\$x1 은 x 라는 데이터에서 x1 변수값들을 달라는 의미이다. 아까 열의 이름을 x1, x2, xs3로 붙였었으니 1열에 대해 리턴해준다.

mean(x$x1) # 평균
sd(x$x1) # 편차
summary(x) # 데이터 x 의 요약통계량

# 2. 벡터와 행렬의 연산

A%*%B 는 행렬 곱
A*B 는 그냥 곱

1. 전치행렬(transpose) 구하기
   m2<-matrix(1:6, ncol=3)
   m2

# transpose of m2

tm2<-t(m2)
tm2

2. Determinant(det) 구하기

# determinant of matrix,

Det 는 NxN 행렬만 구할 수 있음. (공부하자)
d1<-matrix(1:4, nrow=2, byrow=T)
det(d1)

3. 역행렬(inverse) 구하기(solve)
   d1_inv<-solve(d1)
   d1_inv
   즉, 행렬 \* 역행렬 = 단위행렬 됨.

# d1\*inv(d1)=identity matrix

d1%\*%d1_inv

4. 역행렬을 이용한 방정식 해 구하기(solve)
   3x+2y = 8, x+y =2 라는 방정식의 해를 구해보자.

#solve equation

# 3x+2y=8, x+y=2

# matrix a, b

a <- matrix(c(3,1,2,1),nrow=2,ncol=2)
b <- matrix(c(8,2),nrow=2,ncol=1)

# x=4, y=-2

solve(a,b) # 해가 나온다. 이 경우, 계산이 가능한 형태만 넣을 수 있다. 2x2, 2x1 이 아니라 2x2, 1x2 이면 안된다.

5. 고유치(eigenvalue)와 고유벡터(eigenvector)

# example for eigen value and eigen vector

# already centered matrix

x<-matrix(c(-3,-2,0, 1, 2, 2, -3, -3, 0, 2, 2, 2, 5,7,4,0,-5,-11), nrow =6, ncol=3)
x
dim(x)

요렇게 넣으면, x 는 관측치가 6개고 변수가 3개인 데이터임을 의미한다.

# centered matrix (x-mean(x)), not scaled

# x<-scale(x, scale=F)

# eigen value and eigen vector

e1<-eigen(t(x)%_%x) # t(x)%_%x 는 공분산 행렬이라고 한다. 뭔지 모름. 3X3의 아이젠 벡터와 벨류를 값을 구함.
e1

6. 함수 만들기
   square<-function(x){
   return(x\*x)
   } # function 을 생성함.
   square(9)
   square(1:3)

특정 함수를 보려면 함수 이름 치고 실행해봐라
정의가 나온다.

> round
> function (x, digits = 0) .Primitive("round”)

7. 반복문
   for (i in 1:10) {
   if (!i %% 2){
   next
   }
   print(i)
   }

# y는 초기값 정해줘야 한다. i는 1:10 이라는 범위의 로컬 변수를 잡아주는 거고, y는 while 조건문에 넣어줘야 하는 거라.

y=0
while(y <5){ print( y<-y+1) }

# 3. R 데이터 구조(생성, 추출)

1. 데이터 불러들이기

# set working directory

# change working directory

setwd("/Users/admin/project/postechx_r/week3_1")

# check the current working directory

getwd()

# 1. Read csv file : brain weight data

brain<-read.table(file="brain2210.csv", sep=",", na=" ", header=TRUE)

b1<-read.csv("brain2210.csv")

# 엑셀 파일도 csv 로 추출해서 읽는게 편함. txt 는 귀찮음.

head(brain)
dim(brain)

2. attach 사용 : 데이터의 변수 추출
   attach(데이터 이름) : 현재 세션에서 나오는 변수들은 그 데이터의 변수로 인식한다는 의미.

그 데이터를 현재 세션으로 고정한다!

# to get frequency of male and female (brain data)

table(brain\$sex)

# using the command 'attach'

attach(brain)

# get frequency of male and female

table(sex)

# histogram of brain weight

# hist(brain\$brainwt)

hist(wt)

detach(brain)

3. subset : 데이터 추출(데이터 이름, 조건)
   setwd("/Users/admin/project/postechx_r/week3_2")

brain<-read.csv("brain2210.csv")
head(brain)

attach(brain)

# subset with female

# brainf<-subset(brain, sex=='f') after attach(brain)

brainf<-subset(brain, sex=='f’)

# brainf 는 sex가 f인 조건의 데이터 부분집합

mean(brainf$wt) # brainf 의 부분집합에서 wf 라는 데이터의 평균
sd(brainf$wt) # brainf 의 부분집합에서 wf 라는 데이터의 편차

# subset with wt<1300

brain1300<-subset(brain,brain\$wt<1300)

4. aggregate : 그룹별 요약 통계치 구하기
   aggregate(변수~그룹, 데이터, 함수)

# 'aggregate'for statistics by group

aggregate(wt~sex, data=brain, FUN=mean)
aggregate(wt~sex, data=brain, FUN=sd)

5. 추출한 데이터로 그룹별 히스토그램

# histogram for female and male

# 2\*2 multiple plot

par(mfrow=c(2,2))
brainf<-subset(brain,brain$sex=='f') 
hist(brainf$wt, breaks = 12,col = "green",cex=0.7, main="Histogram (Female)" ,xlab="brain weight")

# subset with male

brainm<-subset(brain,brain$sex=='m') 
hist(brainm$wt, breaks = 12,col = "orange", main="Histogram with (Male)" , xlab="brain weight")

# 그런데 위에서는, hist의 x 축과 y축이 다름.

따라서, x축과 y축의 길이를 맞추고, 라벨을 붙여줘야 한다. 아래와 같이 맞춘다.

# histogram with same scale

hist(brainf$wt, breaks = 12,col = "green",cex=0.7, main="Histogram with Normal Curve (Female)" , xlim=c(900,1700),ylim=c(0,25), xlab="brain weight")
hist(brainm$wt, breaks = 12,col = "orange", main="Histogram with Normal Curve (Male)" , xlim=c(900,1700), ylim=c(0,25),xlab="brain weight”)

6. 데이터 내보내기

# 로우 번호 따로 안 매기고, 데이터 내보내기

# export csv file - write out to csv file

write.table(brainf,file="brainf.csv", row.names = FALSE, sep=",", na=" ")

write.csv(brainf,file="brainf.csv", row.names = FALSE)

# export txt file

write.table(brainm, file="brainm.txt", row.names = FALSE, na=" ")

7. 데이터의 전체 구조 파악하기 : str(데이터 이름)
   데이터 프레임은 데이터의 변수의 정보까지 담고 있는 데이터.

Str(car)
'data.frame': 398 obs. of 9 variables:
$ mpg    : num  18 15 18 16 17 15 14 14 14 15 ...
 $ cyl : int 8 8 8 8 8 8 8 8 8 8 ...
$ disp   : num  307 350 318 304 302 429 454 440 455 390 ...
 $ hp : num 17 35 29 29 24 42 47 46 48 40 ...
$ wt     : int  3504 3693 3436 3433 3449 4341 4354 4312 4425 3850 ...
 $ accler : num 12 11.5 11 12 10.5 10 9 8.5 10 8.5 ...
$ year   : int  70 70 70 70 70 70 70 70 70 70 ...
 $ origin : int 1 1 1 1 1 1 1 1 1 1 ...
\$ carname: Factor w/ 305 levels "amc ambassador brougham",..: 50 37 23

8. 데이터의 요약통계치(빈도 구하기) : table(데이터 이름)
   요약통계치는(빈도 구하기) 범주형 변수일때만 가능하다.
   Attach(데이터 이름): 현재 세션에서 나오는 변수들은 그 ‘데이터’의 변수들로 인식. -> 데이터 이름을 안 써줘도 됨!
   attach(car)

# frequency

table(origin)
table(year)

9. 여러 통계치 한번에 구하기 : apply(변수리스트, (1=row, 2=col), Function)

# mean of some variables

apply (car[, 1:6], 2, mean) # car 데이터의 1부터 6까지의 데이터들의 열 평균을 줘라.

10. 막대 그래프 : barplot(count)
    빈도수(table)를 뽑아서 넘겨주자.

# barplot using frequency

freq_cyl<-table(cyl)
names(freq_cyl) <- c ("3cyl", "4cyl", "5cyl", "6cyl",
"8cyl”) # 행에 이름 넣어줌
barplot(freq_cyl)
barplot(freq_cyl, main="Cylinders Distribution”) # 제목 넣어줌

11. 히스토그램 : hist(변수이름, main=“제목”)

# histogram of MPG

hist(mpg, main="Mile per gallon:1970-1982",
col="lightblue”)

12. 3D 산점도 : scatterplot3d ( 변수이름, …, main =“제목”)
    요 패키지 쓸 때는 항상 library를 잡아줘야 함.

# scatterplot3d

# install.packages("scatterplot3d")

library(scatterplot3d)

scatterplot3d(wt,hp,mpg, type="h", highlight.3d=TRUE,
angle=55, scale.y=0.7, pch=16, main="3dimensional plot for autompg data")

13. 벡터화 요약치 : lapply(변수리스트, Function)

# apply a function over a list

lapply (car[, 1:6], mean)

a1<-lapply (car[, 1:6], mean)
a2<-lapply (car[, 1:6], sd)
a3<-lapply (car[, 1:6], min)
a4<-lapply (car[, 1:6], max)
table1<-cbind(a1,a2,a3,a4) # 칼럼 기준으로 다 연결해주는 함수!
colnames(table1) <- c("mean", "sd", "min", "max”) # 칼럼 네임 레이블링
table1

14. Excel 파일에 여러 worksheet이 있을 때

# 1. several sheets in Excel file

install.packages("readxl")
library(readxl)

mtcars1 <- read_excel("/Users/hyunjoonpark/project/postechx_r/week3_4/mtcarsb.xlsx",
sheet = "mtcars")
mtcars1 <- read_excel("/Users/hyunjoonpark/project/postechx_r/week3_4/mtcarsb.xlsx",
sheet = 1)
head(mtcars1)

brain1<-read_excel("/Users/hyunjoonpark/project/postechx_r/week3_4/mtcarsb.xlsx",
sheet = "brain")
head(brain1)

brainm<-read_excel("/Users/hyunjoonpark/project/postechx_r/week3_4/mtcarsb.xlsx",
sheet = 2)
head(brainm)

15. SPSS, SAS, ODBC 데이터 패키지
    RStudio 오른쪽 밑에 보면 packages 있는데, 거기서 foreign 패키지 눌러서 설명 보고, 옆에 체크 보면 라이브러리 꽂는거니까 써봐라.
    이거 외에도 다양한 읽기용 라이브러리는 검색하면서 사용하길

# 2. ODBC data import- STATA, Systat, Weka, dBase ..

install.packages("foreign")
library(foreign)

# 3. Reading SAS data file

install.packages("sas7bdat")
library(sas7bdat)

b1<-read.sas7bdat("brain.sas7bdat")
head(b1)
str(b1)

16. SQL 서버에서 R로 데이터 불러들이기.
    RODBC 패키지 씀. 설정하려면 다 다를 것이다. 나중에 공부하고 적용시켜라.

# 4. Reading from SQL database

# by Anders Stockmarr, Kasper Kristensen

# reading data from SQL databse

install.packages("RODBC")
library(RODBC)

connStr <- paste(
"Server=msedxeus.database.windows.net",
"Database=DAT209x01",
"uid=RLogin",
"pwd=P@ssw0rd",
"Driver={SQL Server}",
sep=";"
)

conn <- odbcDriverConnect(connStr)

#A first query

tab <- sqlTables(conn)
head(tab)

#Getting a table

mf <- sqlFetch(conn,"bi.manufacturer")
mf

close(conn)

# 4. R 그래픽 : 히스토그램

데이터 시각화 : 정보의 요약된 형태를 그래프로 전달하여 새로운 사실(인과관계)를 발견할 수 있다.

1. 데이터의 분포

- 히스토그램 :1차원 (univerariate, 일변량)
- 상자그림(Boxplot) : 1차원 (데이터의 분포를 파악)
- 막대그림(Bar plot): 1차원(범주형 데이터의 빈도 분포)
- 파이 차트(Pie chart) : 1차원(각 범주별 비율을 그림으로)
- 산점도(scatterplot) : 2차원 (x와 y간의 관계를 해석)
  이외의 그래프 형태는 https://www.data-to-viz.com/#histogram 참고.

2. 히스토그램 (색과 제목) : hist(변수이름, col=“색이름”, main=“제목”)

# 1-2. histogram with color and title, legend

hist(wt, breaks = 10, col = "lightblue", main="Histogram of Brain weight" , xlab="brain weight”) # breaks는 자르는 개수

# see rgb values for 657 colors, choose what you like

colors()

# select colors including "blue"

grep("blue", colors(), value=TRUE)

3. 밀도함수 그려보기
   해당하는 분포의 밀도를 표현하는 방법이 있다.

# 1-3. fit function (find density function)

par(mfrow=c(1,1))
d <- density(brain\$wt)
plot(d)

4. 그룹별 히스토그램 (동일한 x축, y축 범위) : xlim, ylim
   동일한 스케일을 맞추기 위해 각 부분집합의 min, max 값을 파악하고 하는 게 좋다.

par(mfrow=c(2,1)) 은 그래프화면의 분할을 행(row)는 2행으로, 열은 1열로 하라는 의미
2,2 를 하면 4개의 그래프를 만들 수 있겠지~~~

# 1-4. histogram with same scale

# multiple plot

par(mfrow=c(2,1))
brainf<-subset(brain,brain$sex=='f') 
hist(brainf$wt, breaks = 12,col = "green", xlim=c(900,1700),ylim=c(0,20),cex=0.7, main="Histogram with Normal Curve (Female)", xlab="brain weight")

brainm<-subset(brain,brain$sex=='m') 
hist(brainm$wt, breaks = 12,col = "orange",xlim=c(900,1700),ylim=c(0,20), main="Histogram with Normal Curve (Male)", xlab="brain weight")

# go back to default graphic setting

#default.parameters <- par(no.readonly = TRUE)
#par(default.parameters)

# op <- par(no.readonly = TRUE)

# par(mfrow=c(1,1), mar=c(4,4,2,2))

# hist(wt)

# par(op)

5. 상자그림 (Boxplot, 1차원)

par(mfrow=c(1,2))

# 2-1 boxplot for all data

boxplot(wt, col=c("coral"))

# 2-2 boxplot by group variable (female, male)

boxplot(wt~sex, col = c("green", "orange”))

# 2-3 horizontal boxplot

par(mfrow=c(1,1))
boxplot(brain$wt~brain$sex, boxwex=0.5, horizontal=TRUE, col = c("grey", "red”))

박스의 width, 폭을 조정할 수 있다. ㄹㅇ 폭만 다름.

# 2-4 box width boxwex (width of box)

par(mfrow=c(1,2))
boxplot(brain$wt, boxwex = 0.25, col=c("coral"),  main="Boxplot for all data")
boxplot(brain$wt, boxwex = 0.5, col=c("coral"), main="Boxplot for all data")

기술통계치 관측치수(n) 넣기.

# 2-5 add text (n) over a boxplot

par(mfrow=c(1,2))
a<-boxplot(brain$wt~brain$sex, col = c("green", "orange”))
text(c(1:nlevels(brain$sex)), a$stats[nrow(a$stats),]+30, paste("n = ",table(brain$sex),sep="”))  # 글자를 a라는 boxplot에 넣겠다. a의 30 보다 높은 위치에 두겠다. Table(brain$sex)는 f, m 77 , 108 로 나오는 범계형 데이터의 통계치니까
77, 108이 순차적으로 나오겠지.

# example : add text (standard deviation) over a boxplot

brainf<-subset(brain,brain$sex=='f')
brainm<-subset(brain,brain$sex=='m')

sdout<-cbind(sd(brainf$wt),sd(brainm$wt))
b<-boxplot(brain$wt~brain$sex, col = c("green", "orange"))
text(c(1:nlevels(brain$sex)), b$stats[nrow(b$stats),]+30, cex=0.8, paste("sd = ",round(sdout, 2),sep="") )

6. 막대그림 (Barplot) : barplot(변수빈도, col=c(“색1”,”색2”, ,,,,)

# use autompg data (lec3_3.R)

car<-read.csv("autompg.csv")
head(car)

attach(car)

# 3. bar plot with cyliner count (lec3_3.R)

# par(mfrow=c(1,1))

table(car\$cyl) # cyl의 빈도수를 freq_cyl에 저장한다.
freq_cyl<-table(cyl)
names(freq_cyl) <- c ("3cyl", "4cyl", "5cyl", "6cyl",
"8cyl”) # 해당 빈도수에 이름을 달아준다.
barplot(freq_cyl, col = c("lightblue", "mistyrose", "lightcyan",
"lavender", "cornsilk”))

7. 파이차트 : pie(변수빈도, labels=c(“라벨1”,”라벨2”, ,,,,)

# 4. pie chart

# You can also custom the labels:

freq_cyl<-table(cyl)
names(freq_cyl) <- c ("3cyl", "4cyl", "5cyl", "6cyl", "8cyl")

# 4-1 pie chart

pie(freq_cyl)
pie(freq_cyl, labels=c("test","test2”))

# 4-2 pie chart clockwise

pie(freq_cyl, labels = c("3cyl", "4cyl", "5cyl", "6cyl","8cyl"),
clockwise = TRUE)

# 4-3 pie chart of subset

# subset with cylinder (4,6,8) - refresh creating subset data lec3_2.R

car1<-subset(car, cyl==4 | cyl==6 | cyl==8)
table(car1\$cyl)

freq_cyl1<-table(car1\$cyl)
pie(freq_cyl1, labels = c("4cyl","6cyl","8cyl"),
clockwise = TRUE)
)

8. 산점도(x와 y의 관계를 보여주는 2차원 그래프) : plot(x,y)
   plot -> 2차원 그래프로 띄워줌.

# 5-1 simple plot from lec1.r

par(mfrow=c(1,1))
x2<-c(1,4,9)
y2<-2+x2
plot(x2, y2)

# 5-2 another simple plot

par(mfrow=c(2,1))
x<-seq(0, 2*pi, by=0.001)
y1<-sin(x)
plot(x,y1, main="sin curve (0:2*pi)")

y2<-cos(x)
plot(x,y2,main="cosine curve (0:2\*pi)" )

어떤 것이 연비가 낮은지 알고 싶다. 예측하기 위해서는 시각화해서 분석해보는 게 좋다.

# scatterplot of autompg data (lec3_3.R)

# 5-3 autompg data (relationship between wt and mpg)

par(mfrow=c(2,1))
plot(wt, mpg)
plot(hp, mpg)

결과를 해석하는 것도 중요하다.

# 5-4 scatterplot coloring group variable

par(mfrow=c(2,1), mar=c(4,4,2,2))
plot(disp, mpg, col=as.integer(car$cyl))
plot(wt, mpg,  col=as.integer(car$cyl)) # cylinder 마다 다른 색으로 표현해줘

9. Subset 데이터를 활용한 산점도 표시

# 5-5 Conditioning plot : seperating scatterplot by factor(group) variable

car1<-subset(car, cyl==4 | cyl==6 | cyl==8) # 4,6,8 실린더의 서브셋 데이터

coplot(예측하고자 하는 변수/변하는 원인인 변수 | factor) # Conditional plot
Factor 마다 다르게 구간을 나눠서 보여주게 된다.
coplot(car1$mpg ~ car1$disp | as.factor(car1\$cyl), data = car1,
panel = panel.smooth, rows = 1)

-> disp 에 따라 mpg 값이 변화하는 걸 car의 cyl 별로 분류해서 보여줘. 해당 데이터는 car1에서 가져가고

?? Pairwise scatterplot을 그리려면 아래와 같이 하란다.
배기량에 따라 변화하는? 뭐 그런 걸 보여주는듯.

# 5-6 cross-tab Plot to see how explanatory variables are related each othe

pairs(car1[,1:6], col=as.integer(car1\$cyl), main = "autompg")

10. 최적 적합 함수 추정(선형회귀모형, 비선형회귀모형)
    Lm(y변수~x변수) : LinearModel 의 약자
    Abline : add line (선을 추가하는 함수)

# 5-7 scatterplot with best fit lines

par(mfrow=c(1,1))
plot(wt, mpg, col=as.integer(car\$cyl), pch=19) # pch=19는 채워진 동그라미를 의미

# best fit linear line

abline(lm(mpg~wt), col="red", lwd=2, lty=1) # linear model인데 mpg를 설명하는 독립변수가 wt라는 모델의 선을 그려줘요. lwd는 선의 굵기, lty는 선의 타입을 의미.

# lowess : smoothed line, nonparmetric fit line (locally weighted polynomial regression)

lowess : locally-weighted polynomial regression
lines(lowess(wt, mpg), col="red", lwd=3, lty=2)
help(lowess) # 선형 아니고, 비선형이 가장 최적일 것 같다. wt를 독립변수로 한 적합식을 보여달라.

11. 그래픽과 레이아웃
    그래프 종류 : plot, barplot, boxplot, hist, pie, persp()
    그래프 구성시 조정사항 : 점, 선의 종류, 글자 크기, 여백조정
    점 그리기 : points()
    선 그리기: lines(), abline, arrows()
    문자 출력: text()
    좌표축 : axis()
    격자표현 : grid()

12. Par()
    그래프의 출력을 조정, 그래프 화면의 분할, 마진, 글자크기, 색상등 설정
    Pty =“s” (x축과 y축을 동일 비율로 설정 square) || “m” (최대 크기로 설정, maximal)
    legend= c(“name1”, “name2”) # 범례
    bty=“o” (box type 그래프의 상자모양을 설정) o, I, 7, c, u
    Pch=1(default) point character (1=동그라미, 2=세모, … 19=채운동그라미)
    lty=1(default) line type (1=직선, 2=점선)
    mar = 아래, 왼쪽, 위쪽, 오른쪽
    cex= 글자크기

par(mfrow=c(2,2))
plot(wt, mpg)
plot(disp, mpg)
plot(hp, mpg)
plot(accler, mpg)

# top 1 plot, bottom 2 plot

(m <- matrix(c(1, 1, 2, 3), ncol = 2, byrow = T))
11
23
이라는 행렬이므로, 그래프를 저렇게 표시해준다.

layout(mat = m)
plot(car$wt, car$mpg, main = "scatter plot of autompg", pch = 19, col = 4)
hist(car$wt)
hist(car$mpg)

13. abline()
    직선 그려주기
    h=y축 위치
    v=x축 위치
    col=column 이름

14. abline(절편값, 기울기값, lty=1, lwd=1,col=colname)
    수직선이 아닌 기울기 있는 직선 그리기

# y = a + bx

abline(a = 40, b = -0.0076, col="red")

# linear model coefficients, lty (line type), lwd (line with)

# linear model (mpg=f(wp))

z <- lm(mpg ~ wt, data = car)
z
abline(z, lty = 2, lwd = 2, col="green")

15. Legend(x축위치, y축위치, legend= 범례라벨, pch=1, col=c, lty=1)

# legend

# scatterplot coloring group variable (lec4_3.R)

par(mfrow=c(1,1), mar=c(4,4,4,4))
plot(wt, mpg, col=as.integer(car\$cyl))
labels = c("3cyl", "4cyl", "5cyl", "6cyl","8cyl")
legend(4000, 45, legend = labels, pch = 1, col =c(3,4,5,6,8), lty =1)

legend의 위치는 그리면서 조정해라.
