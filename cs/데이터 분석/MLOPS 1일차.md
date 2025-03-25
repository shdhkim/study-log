# NLTK (Natural Language Toolkit)

NLTK(Natural Language Toolkit)는 파이썬(Python)에서 자연어 처리를 쉽게 할 수 있도록 도와주는 대표적인 오픈 소스 라이브러리입니다.  
텍스트 전처리, 형태소 분석, 품사 태깅, 구문 분석, 의미 분석 등의 작업을 수행할 수 있는 도구가 함께 제공됩니다.

---

# Matplotlib

Matplotlib은 파이썬의 가장 기초가 되는 시각화 라이브러리입니다.

---

# Seaborn

Seaborn은 Matplotlib을 기반으로 좀 더 고급스럽고 직관적인 시각화 기능을 제공하는 라이브러리입니다.

---

# RMSE (Root Mean Squared Error)

RMSE (Root Mean Squared Error)는 평균제곱근오차라고도 불리며,  
회귀 분석 또는 예측 모델에서 모델의 예측값과 실제값 간의 오차를 측정할 때 흔히 사용하는 지표입니다.  
예측값과 실제값 간의 차이(오차)를 제곱한 뒤 평균을 내고, 그 결과에 루트(제곱근)를 취한 값입니다.

---

# 분류 모델 평가지표

## 정밀도 (Precision)
- 모델이 Positive라고 예측한 것 중 실제로 Positive인 비율

## 재현율 (Recall)
- 실제 Positive 중에서 모델이 Positive로 예측한 비율

## 특이도 (Specificity)
- 실제 Negative를 정확히 Negative라고 예측한 비율

## 거짓 긍정율 (False Positive Rate)
- 실제가 Negative인데 모델이 잘못해서 Positive로 예측한 비율

## 정확도 (Accuracy)
- 전체 예측 중 정답을 맞춘 비율

## F1-Score
- 정밀도와 재현율의 조화평균
- 한쪽 지표만 높지 않고 두 지표가 균형감 있게 높은 모델을 찾을 때 유용

## AUC (Area Under the ROC Curve)
- ROC 곡선 아래 면적을 의미하며, 1에 가까울수록 좋은 성능을 뜻합니다.