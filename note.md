Related Work

Optical Flow as Energy Minimization

    Optical Flow 被視為是energy minimization的問題，使data term以及regularization term之間達到平衡

HS 把 optical flow 用 variational framework 視為 continuous optimization problem，並能夠做到估計 dense flow field
BA 把 optical flow 利用具 robust 的估計 framework 處理過度平滑以及雜訊敏感度
TV-L1 使用 L1 data term and total variation regularization 取代原先的 quadratic penalties，此方法能處理 motion discontinuities and was better equipped to handle outliers(能更好地處理極端值).

像這樣連續的公式維持了在每次迭代中光流法的單個估計，確保目標函式是平滑的(比較好訓練)，使用一階泰勒估計對 data term 建模，不過這個作法，雖然有效，但只適合在小
位移，為了 handle 更多不同大小的位移，這篇論文使用 coarse-to-fine 策略，其中包括圖像金字塔，其金字塔的設計在於低分辨率下估計大位移，然後在高分辨率下估計小位移。

困難點：miss small fast-moving objects and have difficulty recovering from early mistakes

    continuous methods:
