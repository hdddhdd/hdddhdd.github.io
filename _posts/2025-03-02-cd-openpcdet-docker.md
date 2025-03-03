---
layout: post
title: "[코드 리뷰] OpenPCDet Docker로 실행하기"
subtitle: 3D Object Detection
gh-repo: open-mmlab/OpenPCDet
gh-badge: [star, fork, follow]
tags: [3D Object Detection]
comments: true
mathjax: true
author: Heejung Choi
---

# OpenPCDet 이란?
OpenPCDet github에는 다음과 같이 나와있다. 

OpenPCDet is a clear, simple, self-contained open source project for LiDAR-based 3D object detection.

OpenPCDet을 활용하여 KITTI, Waymo, nuScenes 등과 같이 Lidar-based 데이터셋에 대해 3D Object Detection을 수행할 때 기존 모델들을 쉽게 테스트 해볼 수 있다. 


# OpenPCDet을 Docker에서 활용해보자!

## 1. Docker 이미지 다운로드
```bash
sudo docker pull nvidia/cuda:12.1.0-cudnn8-devel-ubuntu22.04
```
## 2. Docker 실행
GPU 전체로 docker를 실행하고 싶다면, 
```bash
sudo docker run --gpus all -it --name openpcdet nvidia/cuda:12.1.0-cudnn8-devel-ubuntu22.04
```

GPU를 분리해서 docker를 실행하고 싶다면,
```bash
sudo docker run --gpus '"device=0"' -it --name openpcdet_gpu0 nvidia/cuda:12.1.0-cudnn8-devel-ubuntu22.04

sudo docker run --gpus '"device=1"' -it --name openpcdet_gpu1 nvidia/cuda:12.1.0-cudnn8-devel-ubuntu22.04
```

이렇게 실행하면 된다 :>

{: .box-note}
[에러] docker: Error response from daemon: could not select device driver "" with capabilities: [[gpu]]. 

[해결방법]
```
# nvidia-container-toolkit 설치
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
   
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit

# 도커 재시작
sudo systemctl restart docker
```
