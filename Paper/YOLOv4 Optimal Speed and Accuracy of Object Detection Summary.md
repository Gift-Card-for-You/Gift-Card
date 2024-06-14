> **YOLOv4 VS YOLOv5**
> 
> 2020년 4월에 발표된 YOLOv4는 SPP와 PAN 등의 기술이 적용되어 더 정확한 Object Detection과 높은 속도를 제공한다. 
> 
> 2개월 뒤, 2020년 6월 Object Detection 정확도 10%이상 향상, 더 빠른 속도와 작은 Model 크기를 가진 YOLOv5이 발표되었다. 
> 
> YOLOv5은 Paper 없이 Code만 공개 되었기 때문에 Paper Review는 YOLOv4, Code는 YOLOv5에 대한 내용이다. 

> **YOLOv4**
> 
> YOLOv4는 기존 YOLO에 있던 작은 크기의 Object Detection 문제를 해결하고자 했다. 
> 
> 다양한 작은 Object Detection을 하기 위해 Input 해상도를 크게 사용했다. 
> 
> 기존 224 X 256등의 해상도를 512 X 512로 증가시켰고, Receptive Field를 물리적으로 키우기 위해 Layer 수를 늘렸다. 
> 
> 또한 하나의 Image에서 다양한 종류, 크기의 Object를 동시에 검출하기 위해서는 높은 표현력이 필요한 것을 고려해 Parameter 수를 키웠다. 
> 
> 정확도를 높이면서 발생하는 속도 문제는 Architecture 변경을 통해 해결했다. 
> 
> Feature Pyramid Network와 Path Aggregation Network 기술을 도입하고 
> 
> Cross Stage Partial Connections 기반의 Backbone 연결과 Spatial Pyramid Pooling 등 새로운 Network 구조를 도입해 성능을 향상시켰다. 
> 
> ![1](https://github.com/Heo-Jeong-Eun/Gift-Card/assets/60500256/cc093cae-1ab2-438d-9ab2-ead0c485b2bf)
> 
> 다양한 Dataset을 이용한 Data 증강 기술과 Quantization 기술을 함께 도입해 높은 성능과 낮은 Model 크기를 갖게 되었다. 

> **YOLOv5**
> 
> YOLOv5은 YOLOv4와 마찬가지로 CSPNet을 사용한다.
> 
> 단, BottleneckCSP를 사용해 각 계층의 연산량을 균등하게 분배해서 연산 Bottleneck을 없애고 CNN Layer의 연산 활용을 향상시켰다. 
> 
> 다른 YOLO와 달리 Backbone을 Depth Multiple과 Width Multiple을 기준으로해 크기별로 YOLOv5 S, M, L, X로 나눈다. 
> 
> ![2](https://github.com/Heo-Jeong-Eun/Gift-Card/assets/60500256/176bdede-0f89-4e01-bbb0-64b76d6a1b9e)
> 
> YOLOv5 X는 느리지만 정확도가 가장 높다.  
> 
> 정확도와 속도는 상충관계이기 때문에 모두 만족시키는 것은 어렵다. 
> 
> YOLOv5 S가 가장 빠른 대신 정확도는 떨어지고, YOLOv5 X가 가장 느린 대신 정확도가 높다. 

## Abstract

CNN 정확도를 향상시키는 다양한 Feature가 있다. 

이러한 Feature의 조합을 Large Dataset에서 실제로 Testing 하고 Result를 이론적으로 정당화 시키는 것이 필요하다.

해당 논문에서는 

1. Weighted-Residual-Connections (WRC)
2. Cross-Stage-Partial-connections (CSP)
3. Cross mini-Batch Normalization (CmBN)
4. Self-adversarial-training (SAT)
5. DropBlock Regularization
6. CloU Loss

다음 6가지 일부를 조합해 Tesla V100에서 약 65 FPS의 Real-Time 속도로 MS COCO Dataset에 대해 43.5% AP 출력하는 최적의 결과를 달성했다. 

- **Question, 문제 제기**
    - CNN의 정확도를 높이는 다양한 기능이 있다.
        
        이를 조합해 Large Dataset에서 Testing 하고 Result를 이론적으로 정당화 시킬 필요가 있다. 
        
- **Functional Classification, 기능 구분**
    - 일부 기능은 특정 Model이나 문제에만 적용되며, 일부는 보편적으로 적용이 가능하다.
        
        Batch-Normalization와 Residual-Connections(잔차 연결)은 대부분의 Model에 적용될 수 있다. 
        
- **New Features, 새로운 기능**
    - WRC, CSP, CmBN, SAT, Mish 활성화 등 새로운 기능들을 도입, 이를 조합하여 MS COCO Dataset에서 43.5% AP(평균 정밀도)와 65 FPS의 실시간 성능을 달성했다.

## Introduction

1. **Limitations of CNN-Based Object Detector, 한계**
    - Coverage, 적용 범위
        - 대부분의 CNN-Based Object Detector는 Recommend System에만 적합하다.
            - 빈 주차 공간 탐색은 느리지만 정확한 Model이 필요하고, 차량 충돌 경고는 빠르지만 부정확한 Model이 사용된다.
    - Improve Real-Time Accuracy, 실시간 정확도 향상
        - Real-Time Object Detector의 정확도를 향상시키면 Recommend System 이외에도 독립적인 Process 관리와 Human Input을 감소시킬 수 있다.

2. **The Need for Real-Time Object Detector, 필요성**
    - Accessibility, 접근성
        - 일반적인 GPU에서 Real-Time으로 운영되면 저렴한 가격에 대량 사용이 가능하다.
    - Current Problem, 현재 문제점
        - 최신 신경망은 Real-Time으로 작동하지 않으며, Training에 많은 수의 GPU가 필요하다.

3. **Solution to YOLOv4**
    - Use Single GPU
        - 일반적인 GPU에서 Real-Time으로 작동하며, Training에 단 하나의 GPU만 필요하다.

4. **Goals**
    - Fast Operating Speed, 빠른 작동 속도
        - Production System에서 Object Detector의 빠른 작동 속도를 설계한다.
    - Optimization for Parallel Computations, 병렬 계산 최적화
        - 낮은 계산량 지표, BFLOP보다 Optimization for Parallel Computations를 중점적으로 한다.

5. **Expectation Effectiveness, 기대 효과** 
    - Easy Training and Use
        - 일반적인 GPU를 사용하는 누구나 쉽게 Training, 사용이 가능하다.
    - Achieve Real-Time, High Quality
        - Real-Time으로 고품질의 설득력있는 Object Detection 결과를 얻을 수 있다.

## **Journal Purpose**

기존 GPU에서 실시간으로 작동하는 Object Detector를 만들고자 하며, Single GPU Training만으로도 가능한 Model을 설계하고자 한다. 

가장 정확한 최신 신경망은 Real-Time으로 작동하지 않으며, 대규모 Mini-Batch 크기로 Training하기 위해 많은 수의 GPU가 필요하다. 

이러한 Network는 각 연구소에서만 Test되며 일반 연구자들에게는 매우 비싸다. 

해당 논문에서는 이러한 문제를 해결하기 위해 일반적인 GPU에서 Real-Time으로 작동하는 CNN을 만들고, Single GPU로 Training할 수 있는 Model을 설계했다. 

1. **Developing an Efficient yet Powerful Object Detection Model**
    - Goals
        - 효율적이면서도 강력한 Object Detection Model 개발
    - User
        - 1080 Ti 또는 2080 Ti GPU User
    - Result
        - 빠르고 정확한 Object Detector Training
        - 고성능 GPU가 필요하지 않더라도 많은 사람들이 접근 가능한 GPU로 높은 성능의 Object Detection Model을 Training할 수 있게 되어, 활용도가 크게 증가한다.
        
2. **Validation of the Impact of State-of-the-Art Bag-of-Freeies and Bag-of-Specials Methods, 영향 검증**

    - Bag-of-Freebies
        - Model의 성능을 높이기 위해 추가 비용 없이 사용할 수 있는 기법
        - Data 증강, 정규화 기법이 이에 해당한다.
    - Bag-of-Specials
        - Model의 성능을 높이기 위해 추가 비용이 발생하지만 성능 향상이 확실한 기법
        - 특정 활성화 함수, 손실 함수가 이에 해당한다.
    - Verification, 검증
        - 해당 기법들이 Object Detection Model의 Training 과정에서 미치는 영향을 실험적으로 검증한다.
        - 실제 Training 과정에서 효과를 확인하고, 이를 바탕으로 Model 성능을 최적화 할 수 있다.

3. **Modified in Ways that are Appropriate for Training a Single GPU, 적합한 방법들로 수정**
    - CBN, Cross Mini-Batch Normalization
        - Mini-Batch Cross 정규화 기법으로, Training 과정에서의 안정성을 높이는 방법
    - PAN, Path Aggregation Network
        - 여러 Layer의 Feature를 결합하여 정보를 더욱 효과적으로 통합하는 Network 구조
    - SAM, Spatial Attention Module
        - 공간적 주의 Mechanism으로, 중요한 Featrue Map에 집중하여 Model의 성능을 향상시키는 방법
    - Edit and Optimization, 수정 및 최적화
        - 이러한 최첨단 방법들을 수정하여 Single GPU 환경에서도 효율적으로 Training할 수 있도록 만든다.
    - Meaning, 의의
        - 일반 사용자들도 고가의 다중 GPU 환경이 아닌, Single GPU 환경에서도 높은 성능의 Model을 Training할 수 있도록 하여, 접근성을 크게 향상시킨다.

## Conclusion

YOLOv4 Model은 효율적인 Bag-of-Freebies와 Bag-of-Specials 기법을 포함해 이전 Model들보다 여러 가지 개선 사항을 가지고 있으며 일반적인 Single GPU에서 실행이 가능하다. 

실험 결과, YOLOv4 Model은 속도와 정확도 모두 다른 Object Detector를 능가하며 다양한 실제 응용 프로그램에 적합함을 보여준다. 

Recommend System 뿐 아니라 Human Input을 요구하는 Real-Time Process Management System에도 적합하다. 

Model을 Parallel Computations(병렬 계산)에 최적화함으로서, Standard Hardware에서도 효율적이고 효과적으로 작동하도록 했다. 

1. Efficient Techniques, 효율적인 기법 
    - Bag-of-Freebies
        - 추가 비용 없이 성능을 높일 수 있는 기법
    - Bag-of-Specials
        - 약간의 추가 비용이 발생하지만 확실한 성능을 보장받는 기법
2.  Real-Time Performance, 실시간 성능
    - 일반적인 GPU에서도 뛰어난 성능을 발휘해 넓은 분야에서 사용 가능
3. Variety of Real-World Application, 다양한 응용 분야
    - Recommend System 뿐 아니라 Real-Time Process Management System에도 적합하다.
    - Human Input 최소화하면서 Real-Time으로 정확한 Object Detection 가능
4. Optimization Parallel Computations, 병렬 계산 최적화 
    - Standard Hardware에서도 효율적, 효과적으로 작동하도록 Model을 최적화

## Results & Discussion

<img width="1000" alt="3" src="https://github.com/Heo-Jeong-Eun/Gift-Card/assets/60500256/2cf315f9-866a-4781-a076-25514d8c84cf">

YOLOv4 Model은 최신 Object Detector와 비교했을 때 속도, 정확성에서 모두 우수한 성능을 보인다. 즉, Pareto 최적성 곡선 상에 위치하고 있음을 보여준다. 

서로 다른 Architecture의 GPU를 사용하는 다양한 방법들을 비교하기 위해 YOLOv4 Model을 Maxwell, Pascal, Volta Architecture의 GPU에서 Test 했다. 