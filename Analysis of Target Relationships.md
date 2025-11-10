# Analysis of Target Relationships :

images: 
Width range: 2000 - 2000
Height range: 1000 - 1000
All sizes: {(2000, 1000)}

1/Dry_Green_g

- Mean ≈ 27, median ≈ 21 → slightly right-skewed (skew ≈ 1.75)
- Range: 0 → 158 → high variability
- 15 samples are zero → usually present, but sometimes absent
- Contributes strongly to Dry_Total_g (correlation ≈ 0.9+)

2/Dry_Dead_g

- Mean ≈ 12, median ≈ 8 → right-skewed (skew ≈ 1.8)
- Range: 0 → 84 → moderate variability
- Correlated with Dry_Total_g, moderate correlation with Dry_Green_g
- Only a few zeros (~7%)

3/Dry_Clover_g

- Mean ≈ 6.8, median ≈ 1.5 → highly skewed (skew ≈ 2.8)
- Many zeros (~36%) → often absent in samples
- Correlation with Dry_Total_g strong, but independent pattern when present

4/GDM_g

- Mean ≈ 34, median ≈ 27 → skew ≈ 1.55
- Range: 1 → 158
- Correlated with Dry_Green_g and Dry_Total_g, but shows unique variance

5/Dry_Total_g

- Mean ≈ 46, median ≈ 41 → skew ≈ 1.4
- Almost perfectly equals Dry_Green + Dry_Dead + Dry_Clover
- Can be used as a sanity check for sum of components

Key Observations :

+] Dry_Total_g ≈ sum of Dry_Green_g, Dry_Dead_g, Dry_Clover_g → linear relationship confirmed
+] Dry_Clover_g zeros → special case for modeling (two-stage or log1p recommended)
+] Skewed distributions → models may under-predict high values if not transformed
+] Strong correlations among dry components → helpful for feature engineering