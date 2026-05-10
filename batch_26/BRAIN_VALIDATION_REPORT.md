# batch_26 Phase 3 — Brain Algorithm Validation Report

**Generated:** 2026-05-10T19:49:21.359Z
**Algorithm:** 40/40/20 (meme reactions / food selections / proximity)
**Source:** `src/services/matchService.js#calculateVibeScore` (ported)
**Personas:** 25
**Eligible pairs scored:** 368

## Aggregate accuracy

| Metric | Hit (≥2/3 overlap) | Partial (≥1/3 overlap) |
|---|---|---|
| Predicted top3 vs actual top3 | **4/25 = 16.0%** | 15/25 = 60.0% |
| Predicted bot3 vs actual bot3 | **2/25 = 8.0%** | 17/25 = 68.0% |

> Pass thresholds (advisory): top ≥60% hit-rate signals strong brain alignment; <40% suggests algorithm or test-data gap. Bottom is a softer signal (false-negative tolerant).

## Per-persona breakdown

### persona_01 Maya — eligible=24, top-overlap=1/3, bottom-overlap=2/3
- predicted top3:  persona_03, persona_22, persona_19
- actual top3:     persona_11(61), persona_03(52), persona_08(52)
- predicted bot3:  persona_07, persona_10, persona_24
- actual bot3:     persona_07(14), persona_10(15), persona_22(20)

### persona_02 Devi — eligible=10, top-overlap=1/3, bottom-overlap=0/3
- predicted top3:  persona_05, persona_12, persona_07
- actual top3:     persona_11(42), persona_19(37), persona_05(27)
- predicted bot3:  persona_21, persona_23, persona_24
- actual bot3:     persona_25(15), persona_15(15), persona_09(19)

### persona_03 Theo — eligible=12, top-overlap=1/3, bottom-overlap=0/3
- predicted top3:  persona_01, persona_22, persona_24
- actual top3:     persona_08(57), persona_20(57), persona_01(52)
- predicted bot3:  persona_10, persona_19, persona_17
- actual bot3:     persona_12(19), persona_22(25), persona_16(25)

### persona_04 Zara — eligible=24, top-overlap=0/3, bottom-overlap=1/3
- predicted top3:  persona_01, persona_17, persona_03
- actual top3:     persona_13(70), persona_07(52), persona_15(47)
- predicted bot3:  persona_07, persona_09, persona_10
- actual bot3:     persona_18(15), persona_10(15), persona_20(19)

### persona_05 Arjun — eligible=12, top-overlap=2/3, bottom-overlap=0/3
- predicted top3:  persona_06, persona_02, persona_18
- actual top3:     persona_06(72), persona_18(65), persona_16(47)
- predicted bot3:  persona_09, persona_19, persona_10
- actual bot3:     persona_12(14), persona_22(20), persona_02(27)

### persona_06 Priya — eligible=10, top-overlap=1/3, bottom-overlap=1/3
- predicted top3:  persona_05, persona_25, persona_08
- actual top3:     persona_05(72), persona_21(42), persona_19(39)
- predicted bot3:  persona_07, persona_19, persona_10
- actual bot3:     persona_07(14), persona_11(24), persona_15(29)

### persona_07 Marcus — eligible=12, top-overlap=1/3, bottom-overlap=1/3
- predicted top3:  persona_02, persona_12, persona_15
- actual top3:     persona_02(22), persona_14(20), persona_01(14)
- predicted bot3:  persona_01, persona_22, persona_04
- actual bot3:     persona_12(4), persona_22(9), persona_18(10)

### persona_08 Yuki — eligible=24, top-overlap=2/3, bottom-overlap=1/3
- predicted top3:  persona_06, persona_03, persona_01
- actual top3:     persona_20(80), persona_03(57), persona_01(52)
- predicted bot3:  persona_07, persona_15, persona_19
- actual bot3:     persona_12(5), persona_07(14), persona_13(19)

### persona_09 Khaled — eligible=12, top-overlap=1/3, bottom-overlap=1/3
- predicted top3:  persona_14, persona_25, persona_22
- actual top3:     persona_22(42), persona_01(35), persona_16(35)
- predicted bot3:  persona_02, persona_04, persona_17
- actual bot3:     persona_12(15), persona_02(19), persona_10(20)

### persona_10 Aisha — eligible=10, top-overlap=1/3, bottom-overlap=0/3
- predicted top3:  persona_05, persona_25, persona_15
- actual top3:     persona_05(35), persona_19(35), persona_03(30)
- predicted bot3:  persona_03, persona_04, persona_17
- actual bot3:     persona_07(10), persona_25(20), persona_11(20)

### persona_11 Liam — eligible=12, top-overlap=1/3, bottom-overlap=1/3
- predicted top3:  persona_14, persona_12, persona_22
- actual top3:     persona_01(61), persona_14(52), persona_24(47)
- predicted bot3:  persona_19, persona_18, persona_07
- actual bot3:     persona_22(19), persona_18(20), persona_10(20)

### persona_12 Sana — eligible=24, top-overlap=0/3, bottom-overlap=0/3
- predicted top3:  persona_11, persona_15, persona_02
- actual top3:     persona_10(52), persona_04(40), persona_13(40)
- predicted bot3:  persona_21, persona_23, persona_03
- actual bot3:     persona_07(4), persona_20(5), persona_08(5)

### persona_13 Diego — eligible=12, top-overlap=0/3, bottom-overlap=0/3
- predicted top3:  persona_15, persona_25, persona_04
- actual top3:     persona_24(45), persona_12(40), persona_14(40)
- predicted bot3:  persona_07, persona_17, persona_19
- actual bot3:     persona_18(15), persona_20(19), persona_08(19)

### persona_14 Naomi — eligible=10, top-overlap=1/3, bottom-overlap=1/3
- predicted top3:  persona_11, persona_22, persona_24
- actual top3:     persona_25(62), persona_11(52), persona_15(47)
- predicted bot3:  persona_07, persona_19, persona_17
- actual bot3:     persona_07(20), persona_09(29), persona_21(34)

### persona_15 Tomas — eligible=12, top-overlap=0/3, bottom-overlap=0/3
- predicted top3:  persona_13, persona_12, persona_10
- actual top3:     persona_14(47), persona_24(42), persona_01(40)
- predicted bot3:  persona_07, persona_19, persona_21
- actual bot3:     persona_02(15), persona_22(24), persona_18(25)

### persona_16 Anjali — eligible=10, top-overlap=2/3, bottom-overlap=1/3
- predicted top3:  persona_05, persona_25, persona_06
- actual top3:     persona_21(50), persona_05(47), persona_25(39)
- predicted bot3:  persona_07, persona_09, persona_15
- actual bot3:     persona_07(10), persona_19(20), persona_03(25)

### persona_17 Rio — eligible=24, top-overlap=1/3, bottom-overlap=1/3
- predicted top3:  persona_19, persona_20, persona_04
- actual top3:     persona_04(47), persona_13(47), persona_15(37)
- predicted bot3:  persona_07, persona_09, persona_10
- actual bot3:     persona_05(14), persona_18(19), persona_07(22)

### persona_18 Kavya — eligible=10, top-overlap=1/3, bottom-overlap=1/3
- predicted top3:  persona_05, persona_06, persona_25
- actual top3:     persona_05(65), persona_19(35), persona_21(35)
- predicted bot3:  persona_07, persona_12, persona_15
- actual bot3:     persona_07(10), persona_13(15), persona_11(20)

### persona_19 Ben — eligible=12, top-overlap=0/3, bottom-overlap=1/3
- predicted top3:  persona_17, persona_20, persona_04
- actual top3:     persona_14(40), persona_06(39), persona_08(39)
- predicted bot3:  persona_02, persona_12, persona_22
- actual bot3:     persona_12(14), persona_16(20), persona_01(24)

### persona_20 Hana — eligible=10, top-overlap=0/3, bottom-overlap=1/3
- predicted top3:  persona_08, persona_24, persona_22
- actual top3:     persona_03(57), persona_25(40), persona_05(39)
- predicted bot3:  persona_07, persona_12, persona_19
- actual bot3:     persona_07(14), persona_13(19), persona_11(24)

### persona_21 Rohan — eligible=12, top-overlap=2/3, bottom-overlap=0/3
- predicted top3:  persona_22, persona_24, persona_01
- actual top3:     persona_01(50), persona_16(50), persona_24(45)
- predicted bot3:  persona_07, persona_12, persona_17
- actual bot3:     persona_22(20), persona_10(25), persona_02(27)

### persona_22 Iris — eligible=24, top-overlap=0/3, bottom-overlap=1/3
- predicted top3:  persona_21, persona_24, persona_03
- actual top3:     persona_09(42), persona_04(40), persona_14(40)
- predicted bot3:  persona_07, persona_19, persona_15
- actual bot3:     persona_07(9), persona_11(19), persona_21(20)

### persona_23 Sami — eligible=24, top-overlap=0/3, bottom-overlap=1/3
- predicted top3:  persona_24, persona_22, persona_04
- actual top3:     persona_03(57), persona_25(57), persona_01(52)
- predicted bot3:  persona_07, persona_19, persona_12
- actual bot3:     persona_02(15), persona_10(20), persona_07(20)

### persona_24 Aanya — eligible=10, top-overlap=0/3, bottom-overlap=2/3
- predicted top3:  persona_21, persona_22, persona_03
- actual top3:     persona_25(57), persona_11(47), persona_13(45)
- predicted bot3:  persona_07, persona_15, persona_19
- actual bot3:     persona_07(14), persona_19(29), persona_05(29)

### persona_25 Mateo — eligible=12, top-overlap=0/3, bottom-overlap=1/3
- predicted top3:  persona_21, persona_22, persona_18
- actual top3:     persona_14(62), persona_24(57), persona_01(45)
- predicted bot3:  persona_07, persona_12, persona_19
- actual bot3:     persona_02(15), persona_10(20), persona_12(25)


## Score matrix (compact, top-half + diagonal omitted)

```
       p01 p02 p03 p04 p05 p06 p07 p08 p09 p10 p11 p12 p13 p14 p15 p16 p17 p18 p19 p20 p21 p22 p23 p24 p25
p01    -  33  52  34  34  29  14  52  35  15  61  20  34  40  40  40  24  25  24  52  50  20  52  52  45
p02   --   -  25  --  27  --  22  --  19  --  42  --  25  --  15  --  --  --  37  --  27  --  --  --  15
p03   52  25   -  --  --  34  --  57  --  30  --  19  --  39  --  25  --  30  --  57  --  25  --  40  --
p04   34  30  35   -  24  29  52  19  29  15  42  40  70  40  47  30  47  15  47  19  34  40  42  45  42
p05   34  27  --  --   -  72  --  39  --  35  --  14  --  34  --  47  --  65  --  39  --  20  --  29  --
p06   --  --  34  --  72   -  14  --  30  --  24  --  29  --  29  --  --  --  39  --  42  --  --  --  34
p07   14  22  --  --  --  14   -  14  --  10  --   4  --  20  --  10  --  10  --  14  --   9  --  14  --
p08   52  19  57  19  39  34  14   -  30  30  24   5  19  35  29  29  25  44  39  80  35  29  47  37  40
p09   35  19  --  --  --  30  --  30   -  20  --  15  --  29  --  35  --  30  --  30  --  42  --  30  --
p10   --  --  30  --  35  --  10  --  20   -  20  --  25  --  25  --  --  --  35  --  25  --  --  --  20
p11   61  42  --  --  --  24  --  24  --  20   -  35  --  52  --  35  --  20  --  24  --  19  --  47  --
p12   20  35  19  40  14  19   4   5  15  52  35   -  40  24  30  24  34   9  14   5  30  24  25  29  25
p13   34  25  --  --  --  29  --  19  --  25  --  40   -  40  --  30  --  15  --  19  --  30  --  45  --
p14   --  --  39  --  34  --  20  --  29  --  52  --  40   -  47  --  --  --  40  --  34  --  --  --  62
p15   40  15  --  --  --  29  --  29  --  25  --  30  --  47   -  30  --  25  --  29  --  24  --  42  --
p16   --  --  25  --  47  --  10  --  35  --  35  --  30  --  30   -  --  --  20  --  50  --  --  --  39
p17   24  35  25  47  14  29  22  25  25  25  35  34  47  30  37  24   -  19  37  25  24  34  25  35  32
p18   --  --  30  --  65  --  10  --  30  --  20  --  15  --  25  --  --   -  35  --  35  --  --  --  34
p19   24  37  --  --  --  39  --  39  --  35  --  14  --  40  --  20  --  35   -  39  --  34  --  29  --
p20   --  --  57  --  39  --  14  --  30  --  24  --  19  --  29  --  --  --  39   -  35  --  --  --  40
p21   50  27  --  --  --  42  --  35  --  25  --  30  --  34  --  50  --  35  --  35   -  20  --  45  --
p22   20  20  25  40  20  35   9  29  42  25  19  24  30  40  24  24  34  29  34  29  20   -  29  35  35
p23   52  15  57  42  39  34  20  47  30  20  47  25  42  45  52  35  25  30  35  47  39  29   -  40  57
p24   --  --  40  --  29  --  14  --  30  --  47  --  45  --  42  --  --  --  29  --  45  --  --   -  57
p25   45  15  --  --  --  34  --  40  --  20  --  25  --  62  --  39  --  34  --  40  --  35  --  57   -
```

(numbers = total vibe score; "--" = ineligible due to lookingFor/gender filter; "-" = self)

## Notes + next-step routing

- Bottom-3 accuracy is intrinsically lower because the algorithm doesn't directly model "incompatibility" — it only ranks similarity. Predicted bot3 is more about taste-divergence than a brain-aware "bad match" signal.
- Top-3 misses fall into two categories:
  1. **Geography pull:** proximity contributes 20%; same-city personas tend to dominate top3 even when food/meme overlap is mediocre.
  2. **Looking-for asymmetry:** if persona A predicted persona B in top3 but B isn't matchable to A under lookingFor/gender filter, B is excluded from A's eligible pool — predicted top3 in PERSONAS.json was composed without applying this filter, so some "predicted" matches are unreachable. Filter these out in any retune analysis.
