# 2021-05-31 (K-means clustering)

k-means clustering

k 개의 군집으로 나누는 방법. 각 군집별 가상의 평균 중심점 위치가 나오게 된다. 처음에 랜덤한 위치로 k 개를 잡고, 전체 점들을 그 k개의 점에 최대한 가까운 곳으로 그룹을 할당해준다. 그 다음에 같은 그룹에 묶인 것들끼리 위치의 평균을 구한다. k 그룹을 다 돌면서 가상의 평균점을 다시 잡고, 그 점들을 기준으로 또 전체 점들에게 가장 가까운 그룹을 할당해준다. 이걸 반복하면서 더 이상 평균이 달라지지 않을 때까지 하면 된다.

처음의 initialization 을 어떻게 최적화하는지는 찾지 못하였다. (이게 할당이 어떻게 되느냐에 따라 결과에 크게 영향을 미칠 수도 있을 것 같다.)



k means clustering Matlab 코드

(출처: https://deep-eye.tistory.com/24)

![img](https://blog.kakaocdn.net/dn/UcvMG/btqN2UlFWpX/hyjtiFU1cM8BSdplviQ0y1/img.png)

![img](https://blog.kakaocdn.net/dn/57c0Q/btqN6TGwbbh/0VG1kYhb5xajVO6LYYni20/img.gif)
```matlab
# *********************************************
# Unsupervised Learning - K-Means algorithm
# Deep.I Inc.
# *********************************************

load corner.mat

# 2차원 공간 데이터
X = X;
# 클러스터 개수
k = 4;                       
# 클러스터 위치 초기화 인덱스 선정
rand = randperm(length(X),k);
# 초기 Centroid 값
u = X(rand ,:);
# 학습 횟수
z = 10;

# 학습 시작
for z=1:10
    # 최대값 저장메모리 할당
    C = cell(k,1);
    for j=1:length(X)
        # 거리 구하기
        for i =1:k
            dist(i,1) = norm(X(j,:)-u(i,:));
        end
        # 중심점과 가장 유사도가 높은 데이터를 중심점 클러스터로 할당
        arg = find(dist==min(dist));
        C{arg}(end+1,:) = X(j,:);
    end
   
    for i = 1:k
        cluster = C{i};
        cluster = sum(cluster) ./ sum(cluster~=0,1);
        try
            u(i,:) = cluster;
        end
    end
    
    # 실시간으로 군집 결과 확인하기
    cla; hold on;
    for i = 1: k
        cluster = C{i};
        try
            scatter(cluster(:,1),cluster(:,2),'.')
            scatter(u(:,1),u(:,2),'*r','LineWidth',5)
        end
    end
    pause(2)
    
end
```



K means clustering C++ 코드 

(출처: https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=zzing0907&logNo=220207572654)

```c++
void kmeans() {
 double ** means = new double*[k];
 
for(int i = 0; i < k; i++)
   means[i] = new double[2];
 
////  set new means  ////          초기값은 아무 데이터나 집어넣는다.
for(int i = 0; i < k; i++) 
    for(int j = 0; j < 2; j++)
    means[i][j] = data[i][j]; 

////  clustering  ////             무한루프
for(;;) {
  int flag = 0;

  for(int i = 0; i < num_data; i++) { 
    double min_dis = 9999;
    int min = -1;

    // calculating distances //        거리를 재서 가장 가까운 평균을 구함
    for(int j = 0; j < k; j++) {
      double dis = 0;
      for(int l = 0; l < 2; l++)
        dis += pow(float(data[i][l] - means[j][l]),2);
      if(dis < min_dis) {
        min_dis = dis;
        min = j;
       }
    }
       
    // set groups //      가장 가까운 평균과 같은 그룹에 넣음
    group[i] = min;
  }

  // allocate new memory for calculate //
  double **temp = new double*[k];
  int *count = new int[k];
 
  for(int i = 0; i < k; i++) {
    count[i] = 0;
    temp[i] = new double[2];
    for(int j = 0; j < 2; j++)
      temp[i][j] = 0;
  }
 
  // calculating new means //       새로운 평균을 구하고 평균이 하나라도 다르다면 다시계산
  for(int i = 0; i < num_data; i++) { ​  //  여기서는 flag를 둬서 브레이크 여부를 결정함
    count[group[i]]++;
    for(int j = 0; j < 2; j++)
      temp[group[i]][j] += data[i][j];
     }
    
  for(int i = 0; i < k; i++)
    for(int j = 0; j < 2; j++) {
      temp[i][j] /= count[i];
      if(fabs(temp[i][j] - means[i][j]) > 0.0001) {   //  데이터가 실수라 오차가 있는 것 같아서 오차범위를 0.0001로 둠
        flag++;
        means[i][j] = temp[i][j];
      }
    }

  // break point //
  if(flag == 0)
    break;
}

////  freeing memory  ////
for(int i = 0; i < k; i ++)
  delete means[i];
  delete means;
}
```

