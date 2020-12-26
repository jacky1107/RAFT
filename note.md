## Related Work

### Optical Flow as Energy Minimization

    Optical Flow 被視為是energy minimization的問題，使data term以及regularization term之間達到平衡

    HS 把 optical flow 用 variational framework 視為 continuous optimization problem，並能夠做到估計 dense flow field
    BA 把 optical flow 利用具 robust 的估計 framework 處理過度平滑以及雜訊敏感度
    TV-L1 使用 L1 data term and total variation regularization 取代原先的 quadratic penalties，此方法能處理 motion discontinuities and was better equipped to handle outliers(能更好地處理極端值).

    像這樣連續的公式維持了在每次迭代中光流法的單個估計，確保目標函式是平滑的(比較好訓練)，使用一階泰勒估計對 data term 建模，不過這個作法，雖然有效，但只適合小位移，為了 handle 更多不同大小的位移，這篇論文使用 coarse-to-fine 策略，其中包括圖像金字塔，其金字塔的設計在於低分辨率下估計大位移，然後在高分辨率下估計小位移。

    Coarse-to-fine 策略的困難點：miss small fast-moving objects and have difficulty recovering from early mistakes

    同時 Coarse-to-fine 策略也能保持像 continuous methods 一樣，能夠維持及完善光流法的單個估計
    然而為了使 each local update 都能掌握小位移和大位移的資訊，使用了一個方法 - 分別在每一層金字塔(low resloution and high resolution)，對 all pairs(?)建立 correlation volumes.

    此外，在這邊提到了 update operator 是學著怎麼找到下降方向，而不是使用 subpixel 泰勒估計

    最近，大家試著利用離散化的方法去解決全域性光流法的問題，而這種方法的缺點在於要在龐大的範圍試著找到最佳解(一個 pixel 被成千上萬的合理地配對)

    也因為此問題，有人提出了縮小最佳解範圍的方法 - 利用 feature descriptor 以及最大後驗機率，亦或是使用距離變化

    近幾年來，也有人嘗試使用 NN 當作 feature descriptor 並建構 4D cost volume over all pairs of features.

    接著 DCFlow 利用 Semi-global matching 來尋找最佳解(between different frames)，但這過程是利用 embedding loss between pixels 來訓練 network，只不過這個方法是不可微的

    而此篇論文，嘗試利用 NN 來估計 flow，並且保持整個 end-to-end 過程是可微分的

### Direct Flow Prediction

    此篇論文的目的是訓練出一個神經網路能夠直接預測 optical flow between a pair of frames，並且避開最佳化問題

    Coarse-to-fine 技術被很多人使用，但不同的點是他們維持且提升單張高解析度的光流法流暢度

### Iterative Refinement for Optical Flow

    The main difference of these approaches from ours is that they do not share weights between iterations.

    該篇論文的 update operator uses a modified GRU block which job is converge a fixed flow field easily.

### Learning to Optimize

    該篇論文其中一個特點在於，他們並沒有明確地定義優化目標，反而是使用了大量的更新模塊來模擬一階優化演算法，並在先前所建立好的correlation volumes提取特徵並試著找到更新方向

## Approach

    1. Feature Extraction:
    2. Iterativa Updates:
        該步驟要利用 update operator 從 f0 = 0 開始一步步估計 flow(fn)，在每一次迭代的時候，都會產生並更新方向(delta f)變成fn = df + fn，而這個update operator is trained to perform updates使sequence converges收斂到固定點

    2.1 Initialization:
        初始f = 0，而在這他們先設定好當前後frame有出現occlusion gaps的話則利用nearest neighbor interpolation來填補

    2.2 Inputs:
        correlation features processed by 2 conv layers and also let itself to generate flow features. Additionaly, the input feature map is concatenation of te correlation, flow and context features.

    2.3 Update:
        在更新及迭代的這個步驟他們使用GRU架構，但把FCN換成Conv2d，並嘗試兩種filter: 3x3 以及 垂直水平方向(1x5)，結果發現垂直水平方向的filter可以增加感知範圍且並不會影響model size太多

    2.4 Flow Prediction:
        GRU的輸出hidden state，再經由兩層的conv2d去預測將要更新的delta f，而此時的output flow只有1/8的resolution，但最終還是要upsample to match ground truth to training and evalution.

    2.5 Upsampling:

