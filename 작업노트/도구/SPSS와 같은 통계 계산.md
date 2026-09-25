---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-26
updated: 2026-09-26
projects:
  - "[[프로젝트/개인/논문표/README|논문표]]"
---

# SPSS와 같은 통계 계산

한국 논문이 인용하는 SPSS 값과 맞추려면 **라이브러리 기본값이 SPSS와 다른 곳**을 하나씩 명시적으로 맞추고, 기댓값은 기억이 아니라 scipy·statsmodels 실행 출력에서 가져온다.

## 핵심 정리

| 항목 | SPSS | 흔한 라이브러리 기본값 | 맞추는 법 |
|---|---|---|---|
| Levene 등분산 | 평균 기준(Levene 원래 방식) | `scipy.stats.levene` 기본 `center='median'`(Brown-Forsythe) | `center='mean'` |
| 왜도·첨도 | 편향 보정 G1, 초과첨도 G2 | `scipy.stats.skew`·`kurtosis` 기본 `bias=True` | `bias=False`, 첨도는 `fisher=True` |
| 카이제곱 | Pearson(무보정)을 보고 | `scipy.stats.chi2_contingency` 기본 `correction=True`(자유도 1일 때 Yates) | `correction=False` |
| 표준편차 | n-1 | `numpy.std` 기본 `ddof=0` | `ddof=1` |
| 결측 | 상관 쌍별, 기술통계 변수별, 나머지 목록별 | 분석마다 다름 | 분석별로 명시 |
| Scheffé 사후검정 | 제공 | scipy·statsmodels에 없음 | F_s = 차이² / (MS_within(1/n_i+1/n_j)) / (k-1), p = F 상측(k-1, N-k) |
| VIF | 1/(1-R²_j), 상수 포함 보조회귀 | `statsmodels…variance_inflation_factor(X, i)` | **상수 열을 포함한** X를 넘긴다 |

JS 쪽 p값은 1-cdf 대신 정규화 불완전 베타로 계산하면 작은 p에서도 정밀하다: t 양측 `ibeta(df/(df+t²), df/2, 1/2)`, F 상측 `ibeta(d2/(d2+d1·F), d2/2, d1/2)`.

## 기록

### 2026-09-26 — 정답지는 scipy로 뽑고, 모든 분기를 지나가게 설계한다

- 맥락: [[프로젝트/개인/논문표/README|논문표]] — 8종 분석을 TypeScript(jStat)로 구현하며 SPSS 값 일치를 목표로 했다.
- 배운 것:
  1. **정답지 방식**: 시드 고정 가상 설문(n=200, 역문항 JS3, 결측 5칸, 같은 데이터의 xlsx·UTF-8 CSV·CP949 CSV)을 파이썬으로 만들고 scipy 1.13.1·statsmodels 0.14.6·pingouin 0.5.5로 참조값을 뽑아 `reference.json`에 저장한다. TS 테스트는 결과 타입의 **모든 필드**를 비교한다. 생성기는 `scripts/gen_reference.py`.
  2. **픽스처가 모든 분기를 지나가야 한다.** 처음에 효과를 약하게 잡아 분산분석이 비유의(p=.09)로 나왔고, 그러면 Scheffé 경로는 한 번도 검증되지 않는다. 효과를 키워 분산분석 p=.0002·Scheffé `d>a, d>b`, t검정 유의(p=.019), 교차분석 비유의(p=.757)가 동시에 나오게 했다 — 해석문의 유의·비유의 문장이 둘 다 검증된다.
  3. **기억으로 적은 기댓값은 틀린다.** 테스트에 `2*t.sf(2, 10)`을 0.07346으로 적었는데 scipy 출력은 0.073388이었다. 기댓값은 반드시 실행 출력을 붙여 넣는다.
  4. **jStat `ibeta`의 상대정밀도는 1e-8 수준**이다. 허용 오차를 통계량 상대 1e-9, p값 상대 1e-6으로 나눴다. p는 표에 셋째 자리까지만 나가서 결과에 영향이 없다.
  5. **회귀 특이 판정은 스케일을 맞춘 뒤에 한다.** "피벗 < X'X 대각 최대값 × 1e-10"으로 설계했더니 단위가 크게 다른 변수(원 단위 월소득 + 0/1 더미, 대각 비율 약 1e-13)에서 공선성이 없는데도 에러가 났다. S = diag(1/√a_ii)로 SAS(대각 1)를 만들어 뒤집고 A⁻¹ = S·(SAS)⁻¹·S로 되돌려 해결.
  6. **p 표기 경계**: p=.0497을 셋째 자리로 반올림하면 ".050"인데 별(*)이 붙어, 표는 ".050*" 해석문은 "p<.05"가 된다. 반올림이 .05·.01 경계를 넘으면 자릿수를 늘려 ".0497"로 쓴다.
  7. **회귀 B·S.E.는 최소 셋째 자리.** 둘째 자리면 작은 계수(근무경력 B=0.0231, S.E.=0.0105)가 0.02·0.01로 뭉개져 t=B/SE 관계가 표에서 안 맞아 보인다.
- 근거: nonmunpyo 커밋 `ef9f67d`(정답지·공통 통계 함수), `b6adbc4`(회귀 대각 스케일링), `f712b75`(p 경계 표기), `4209981`(B·S.E. 셋째 자리). `npm test` 270개 통과(2026-09-26).

## 참고 자료

- [scipy.stats.levene](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.levene.html) — 기본 `center='median'`, 평균 기준이 Levene 원래 방식이고 median은 Brown-Forsythe (2026-09-26 확인)
- [scipy.stats.chi2_contingency](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.chi2_contingency.html) — 기본 `correction=True`, 자유도 1일 때만 Yates 적용 (2026-09-26 확인)
