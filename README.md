# Deeplabv3_segmentaion_iou

https://github.com/msminhas93/DeepLabv3FineTuning.git 참고

IoU 점수 산출을 위한 코드 수정 작업 진행

---

- 2022.07.23  [ MAIN TASK : Deeplabv3 reference IoU ]
    - Deeplabv3 reference code에 IoU 점수를 산출하는 코드 작업 진행
        - 참고 코드: Assignment_3_Semantic+Segmentation+finetuning_end.ipynb
        - batch_segmentation_iou : iou 계산 하는 함수
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e4d0c249-9c2d-48c3-9c4f-04cb26cdb757/Untitled.png)
        
    
    - 참고 코드: Assignment_3_Semantic+Segmentation+finetuning_end.ipynb
        - batch_segmentation_iou 함수를 사용하여 iou 점수 계산 진행 코드 참고
            
            ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ee8343b7-4ef8-4f2e-8d84-457e710f01cc/Untitled.png)
            
    - Binary_512x512_Deeplabv3_FineTuning 코드는 prediction과 mask 값을 numpy array 형태로 계산하나, 위 코드는 tenser 형태로 계산 됨 (y_pred: out_target, y_true: label 과 비슷함)
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e81df006-e4cd-40e9-a592-46a09812b2e1/Untitled.png)
        
    - batch_segmentation_iou 함수의 input parameter shape는 [b, c, w, h] 형태이나, Binary_512x512_Deeplabv3_FineTuning 코드의 y_pred와 y_true는 revel()함수를 통해 다차원 배열을 1차원으로 평평하게 재배열 한 상태이며, 참고 코드와 이번 reference코드와는 맞지 않았다.
    - [https://velog.io/@tree_jhk/모든-모양의-IoU-계산하는-방법](https://velog.io/@tree_jhk/%EB%AA%A8%EB%93%A0-%EB%AA%A8%EC%96%91%EC%9D%98-IoU-%EA%B3%84%EC%82%B0%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95) 참고
        - 이미지를 array 형태로 read하여 array 끼리 intersection과 union을 산출하는 방식이 소개되어 있음
        - but.. reference 코드에서는 data가 [b, c, w, h]로 transform 되어있으며, array행렬을 출력해 봤을 때 0과 1사이의 어떠한 값으로 되어 있어 다시 image 형태의 array 형태로 바꿔주는데 어려움이 있어 reference 코드에선 해당 방법으로 구현이 어렵다 판단함…
    - 1차원으로 되어있는 array 형태의 배열을 이용해 intersection과 union을 계산할 수 있는 코드를 직접 구현하기로 함
    - np.logical_and(pred, label) 함수를 통해 intersection를 구하고, np.logical_or(pred, label) 함수를 통해 union을 구하고자 함 // 위 두 함수는 0, 1을 이용하여 or gate, and gate 방식으로 동작함 //  ex) and일 경우 0,0일 때 false / 0, 1일 때 false / 1, 0일 때 false / 1, 1 일 때 true / 를 반환 함
    - and 일 때 true 개수, or 일 때 true 개수를 구해서 iou = (intersection + SMOOTH) / (union + SMOOTH)를 계산 해줌 // SMOOTH: 분자나 분모가 0이 될 경우 무한대의 값 또는 0으로 계산되어 나올 수 있기 때문에 작은 수를 더해 줌으로써 이를 방지함
    - 완성 된 iou 계산 함수
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bcc7085b-7557-4dfa-903a-159b33b642df/Untitled.png)
        
    - array shape, tensor shape, iou 산출 방법, iou 계산식의 위치,  reference code에서 main.py와 trainer.py의 상관 관계, data shape에 따른 계산 방식, 다른 코드에서 인용할 때 변형하는 방법 등… 많은 error를 보았고, 코드에서 각 위치마다 print를 사용해서 출력 값을 찾아보고 실험해봄… 시간이 다소 소요되지만 가장 좋은 방법은 error가 발생한 부분에서 print를 찍어가며 거슬러 올라가는 방법인 듯함…
    
    -
