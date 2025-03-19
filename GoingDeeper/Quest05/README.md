# AIFFEL Campus Online Code Peer Review Templete
- 코더 : 김민호
- 리뷰어 : 이정우


# PRT(Peer Review Template)
- [ ]  **1. 주어진 문제를 해결하는 완성된 코드가 제출되었나요?**
    - 문제에서 요구하는 최종 결과물이 첨부되었는지 확인
        - 중요! 해당 조건을 만족하는 부분을 캡쳐해 근거로 첨부
     

          ![image](https://github.com/user-attachments/assets/f3e7f0fc-1b39-4a1d-a5dd-b2ffeae1b605)

          프로젝트에서 요구하는 90%이상의 정확도가 나왔습니다.

    
- [ ]  **2. 전체 코드에서 가장 핵심적이거나 가장 복잡하고 이해하기 어려운 부분에 작성된 
주석 또는 doc string을 보고 해당 코드가 잘 이해되었나요?**
    - 해당 코드 블럭을 왜 핵심적이라고 생각하는지 확인
    - 해당 코드 블럭에 doc string/annotation이 달려 있는지 확인
    - 해당 코드의 기능, 존재 이유, 작동 원리 등을 기술했는지 확인
    - 주석을 보고 코드 이해가 잘 되었는지 확인
        - 중요! 잘 작성되었다고 생각되는 부분을 캡쳐해 근거로 첨부
     

def compute_expected_label(sample, size_limit=300):
    """
    어노테이션 정보를 기반으로 expected label을 산출합니다.
    조건:
      - 사람이 하나 이상 존재하거나
      - 차량(차, 트럭, 밴, 버스)의 너비 또는 높이가 size_limit(px) 이상이면 "Stop"
      - 그렇지 않으면 "Go"
    """
    image = sample["image"]
    h, w, _ = image.shape
    bboxes = sample["objects"]["bbox"]  # 정규화된 좌표 [ymin, xmin, ymax, xmax]
    types = sample["objects"]["type"]
    
    for bbox, obj_type in zip(bboxes, types):
        # int2str 함수는 KITTI 라벨 인덱스를 문자열로 변환
        label = int2str(int(obj_type)).lower()
        ymin, xmin, ymax, xmax = bbox
        abs_width = (xmax - xmin) * w
        abs_height = (ymax - ymin) * h
        if "person" in label or "pedestrian" in label:
            return "Stop"
        if label in ["car", "truck", "van", "bus"]:
            if abs_width >= size_limit or abs_height >= size_limit:
                return "Stop"
    return "Go"

def test_system_dataset_from_ds_train(ds_test, func, num_samples=100, size_limit=300):
    """
    이미 로드된 ds_train에서 num_samples개의 샘플을 가져와,
    각 이미지의 어노테이션으로 expected label("Stop" 또는 "Go")을 산출한 후
    self_drive_assist 함수의 예측 결과와 비교
    """
    ds_samples = ds_test.take(num_samples)
    
    score = 0
    total = 0
    for sample in tfds.as_numpy(ds_samples):
        expected = compute_expected_label(sample, size_limit=size_limit)
        # self_drive_assist는 이미지 경로를 입력받으므로, 임시 파일에 저장
        img = sample["image"]
        temp_dir = tempfile.gettempdir()
        temp_path = os.path.join(temp_dir, "temp_image.png")
        im = Image.fromarray(img)
        im.save(temp_path)
        
        pred = func(temp_path, size_limit=size_limit)
        print(f"Sample {total+1}: Expected: {expected}, Predicted: {pred}")
        if pred == expected:
            score += 1
        total += 1
    print(f"\nAccuracy: {score/total*100:.2f}% (score: {score}/{total})")

# ds_train은 이미 위에서 불러와진 상태라고 가정
# 예: ds_train = tfds.load("kitti", split="train", shuffle_files=True, download=False)

test_system_dataset_from_ds_train(ds_test, self_drive_assist, num_samples=100)

저하고는 다른 방식으로 해주셔서 참고가 되었습니다.


        
- [ ]  **3. 에러가 난 부분을 디버깅하여 문제를 해결한 기록을 남겼거나
새로운 시도 또는 추가 실험을 수행해봤나요?**
    - 문제 원인 및 해결 과정을 잘 기록하였는지 확인
    - 프로젝트 평가 기준에 더해 추가적으로 수행한 나만의 시도, 
    실험이 기록되어 있는지 확인
        - 중요! 잘 작성되었다고 생각되는 부분을 캡쳐해 근거로 첨부
     
![image](https://github.com/user-attachments/assets/7958bf55-18cf-4cce-ae7c-d4b94f3dd46e)

로컬과 lms로 하는것이 조금 다르구나를 좀 느꼈습니다. 버전차이 때문인 것 같기도하고 그렇습니다.

        
- [ ]  **4. 회고를 잘 작성했나요?**
    - 주어진 문제를 해결하는 완성된 코드 내지 프로젝트 결과물에 대해
    배운점과 아쉬운점, 느낀점 등이 기록되어 있는지 확인
    - 전체 코드 실행 플로우를 그래프로 그려서 이해를 돕고 있는지 확인
        - 중요! 잘 작성되었다고 생각되는 부분을 캡쳐해 근거로 첨부
     
          ![image](https://github.com/user-attachments/assets/03589e1a-01ed-4bc1-ba90-fed8270b61e0)

        
- [ ]  **5. 코드가 간결하고 효율적인가요?**
    - 파이썬 스타일 가이드 (PEP8) 를 준수하였는지 확인
    - 코드 중복을 최소화하고 범용적으로 사용할 수 있도록 함수화/모듈화했는지 확인
        - 중요! 잘 작성되었다고 생각되는 부분을 캡쳐해 근거로 첨부
     
          


# 회고(참고 링크 및 코드 개선)
```
# 리뷰어의 회고를 작성합니다.
# 코드 리뷰 시 참고한 링크가 있다면 링크와 간략한 설명을 첨부합니다.
# 코드 리뷰를 통해 개선한 코드가 있다면 코드와 간략한 설명을 첨부합니다.

마지막에 test 데이터를 통해서 할 생각을 못해봐서 아차 싶었습니다. 잘 참고가 되었습니다. 감사합니다.


```
