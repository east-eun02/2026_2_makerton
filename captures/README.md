# 순찰 기록 (자동 업로드)

로봇 노트북의 `capture_uploader`가 순찰 회차가 끝날 때마다 이 폴더에 올린다. 사람이 직접 고치지 않는다.

## 구조

```
captures/
  index.json                  # 회차 목록과 지점별 판정 요약 (사이트는 이것부터 읽는다)
  baseline_<루트>.txt          # 루트별 기준 회차(정상 화분) 이름
  <회차>/                      # 회차 = 순찰 시작 시각 YYYY-MM-DD_HHMMSS
    result.json               # 그 회차의 전체 기록
    <지점 ID>.png              # 지점에서 찍은 사진 (예: P00.png)
    <지점 ID>_compare.jpg      # 기준 | 이번(정렬) | 변화 표시 3칸 비교 그림 (비교했을 때만)
```

## index.json

```jsonc
{
  "updated": "2026-10-30T00:23:10",
  "baselines": {"demo_real3": "2026-10-29_192945"},
  "laps": [                                   // 최신 회차가 먼저
    {
      "lap": "2026-10-30_002113", "route": "demo_real3",
      "started": "2026-10-30T00:21:13", "finished": "2026-10-30T00:22:50",
      "stops": [
        {
          "id": "P00",
          "status": "ok",                     // ok | nav_failed(이동 실패) | no_image(사진 없음)
          "image": "2026-10-30_002113/P00.png",   // captures/ 기준 경로, 없으면 null
          "compare": {                        // 비교를 안 했으면 null
            "status": "changed",              // changed | unchanged | no_reference | align_failed | no_plant | no_image | error
            "change": "fruit_added",          // changed일 때 가장 큰 변화 종류, 아니면 null
            "count": 3,                       // 그 종류의 개수
            "reference_lap": "2026-10-29_192945",
            "reference_image": "2026-10-29_192945/P00.png",
            "image": "2026-10-30_002113/P00_compare.jpg"
          }
        }
      ]
    }
  ]
}
```

변화 종류: `leaf_loss` 잎 탈락, `fruit_added` 열매 생성(성장 기록), `leaf_color` 잎 색 변화,
`fruit_damaged` 열매 손상, `other` 기타 변화. 덩어리별 위치(bbox)·면적 등 자세한 값은 각 회차의 `result.json`에 있다.

지점의 `final`이 있으면 최종 판정으로 사용한다. `final.verdict`는 `confirmed`(재촬영 확인),
`mismatch`(불일치), `recheck_used`(재촬영으로 판정), `recheck_failed`(재촬영 실패)이며,
`final.changes`와 `final.agreeing_views`에 종류별 개수와 일치한 촬영 방향이 있다.
`views.front/left/right`에는 방향별 사진·비교·촬영 위치가 담긴다. 사진 경로는 모두 `captures/` 기준이다.
`compare`는 최초 사진의 결과이므로 `final`을 대신하지 않는다. 회차의 `home`, `mode`,
`baseline_complete`도 결과 파일과 동일하게 제공한다.

사진 주소 예 (GitHub Pages가 이 저장소를 올린 경우): `https://<팀>.github.io/<저장소>/captures/2026-10-30_002113/P00.png`
