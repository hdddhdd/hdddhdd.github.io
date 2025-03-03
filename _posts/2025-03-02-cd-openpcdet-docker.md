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
OpenPCDet github에는 다음과 같이 나와있습니다!

OpenPCDet is a clear, simple, self-contained open source project for LiDAR-based 3D object detection.

OpenPCDet을 활용하면 KITTI, Waymo, nuScenes 등과 같이 Lidar-based 데이터셋에 대해 3D Object Detection을 수행할 때 기존 모델들을 쉽게 테스트 해볼 수 있습니당!


# OpenPCDet을 Docker에서 활용해보자!

## Docker 이미지 다운로드
```bash
sudo docker pull nvidia/cuda:12.1.0-cudnn8-devel-ubuntu22.04
```
## Docker 실행
GPU 전체로 docker를 실행하고 싶다면, 
```bash
sudo docker run --gpus all -it --name openpcdet nvidia/cuda:12.1.0-cudnn8-devel-ubuntu22.04
```

GPU를 분리해서 docker를 실행하고 싶다면,
```bash
sudo docker run --gpus '"device=0"' -it --name openpcdet_gpu0 nvidia/cuda:12.1.0-cudnn8-devel-ubuntu22.04

sudo docker run --gpus '"device=1"' -it --name openpcdet_gpu1 nvidia/cuda:12.1.0-cudnn8-devel-ubuntu22.04
```

이렇게 실행하면 됩니다요 

[에러]
docker: Error response from daemon: could not select device driver "" with capabilities: [[gpu]]. 

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

## OpenPCDet을 실행하기 위해 이것들을 설치해야 합니다!
먼저 가상환경을 만들고, 필요한 패키지를 설치합시닷
아 그 전에 클론 해오셔야 해요!
```bash
git clone https://github.com/open-mmlab/OpenPCDet.git
```

```bash
apt-get update
apt-get install -y python3 python3-pip git wget
apt-get install -y python3-venv
python3 -m venv openpcdet
source openpcdet/bin/activate
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

pip install -r requirements.txt
pip install spconv-cu120 open3d av2 kornia==0.6.5
```

pcdet 설치를 위해 이것도 해주십시오
```bash
python setup.py develop
```

이것은 데이터 전처리용입니다 :3 저는 kitti 위주로 했기 때문에 kitti로 써놓을게용! github에 각 데이터별 전처리 커맨드는 [여기](https://github.com/open-mmlab/OpenPCDet/blob/master/docs/GETTING_STARTED.md)에 잘 나와있습니다!

```bash
python -m pcdet.datasets.kitti.kitti_dataset create_kitti_infos tools/cfgs/dataset_configs/kitti_dataset.yaml 
```

[에러]
ImportError: libGL.so.1: cannot open shared object file: No such file or directory

[해결방법] apt-get install -y libgl1-mesa-glx  

[에러] 
ImportError: libgthread-2.0.so.0: cannot open shared object file: No such file or directory

[해결방법] apt-get install libglib2.0-0

## 모델 학습
예시는 pointpillar 모델로 들어보겠습니다!
```bash
python train.py --cfg_file cfgs/kitti_models/pointpillar.yaml --workers [worker 개수] --epochs [학습 epoch 수] --extra_tag [별도로 붙일 tag이름, 필수 아님] --batch_size [batch size]
```

아주 쉽죠? 

Openpcdet에서 지원하는 모델이라면 cfg 파일만 다르게 줘서 학습시킬 수 있어요! 

모든 학습 결과는 output 폴더에 저장됩니다! 

extra_tag는 학습을 다양하게 시킬 경우에, 예를 들어서 같은 모델을 학습시켜도 코드를 수정해서 다양하게 학습시키는 경우 등에서 학습한 결과 파일을 구분할 때 좋으니까 필요한 경우에는 extra_tag를 달아주는 것을 추천 와바박 드립니다.

이 외에도 많은 옵션이 있으니까 train.py에서 확인해보십시오!

예를 들면, --max_ckpt_save_num은 default로 30으로 설정되어 있는데, 이 옵션은 최대 몇개의 ckpt(.pth)파일을 저장할 수 있게할지 정하는 옵션입니다! 30으로 두면 만약 80epoch 학습시킬 경우에 51부터 80번째까지밖에 저장이 안됩니다! 원하는 만큼 옵션으로 정해놓는게 좋겠죠?!

## VSCode 상에서 Devcontainer에 내가 만든 컨테이너가 보이지 않는다면!?!
당황하지 마세요.
```bash
sudo usermod -aG docker $USER
```
이것만 해주면 만사 오케이! 오케이입니다

![ok](/assets/img/mansaok.jpg)


혹시 궁금한 점이나 에러가 생긴다면 댓글을 달아주십쇼!

감사합니다!