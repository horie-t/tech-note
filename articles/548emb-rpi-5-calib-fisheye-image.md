---
title: "Raspberry Pi 5に接続した魚眼レンズカメラの画像の歪みを校正する"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["raspberrypi", "opencv", "image"]
published: true
---

以前の記事では、Raspberry Pi 5に[Raspberry Pi用魚眼レンズカメラモジュール（8MP / IMX219）](https://www.switch-science.com/products/8245?variant=42382205223110)を接続して、色ムラ等の補正をして撮影をしました。

このままでは以下のように歪んだ形の画像が撮影されます。

![](/images/rpi-fisheye.jpg =320x)

本記事ではOpenCVのパッケージを使って歪んだ形の画像の歪みを校正します。

## 校正用画像の撮影

[校正用のパターン画像](http://opencv.jp/sample/pics/chesspattern_7x10.pdf)を印刷して、10～20枚ほど撮影します。この時、以下の点について意識して様々な条件で撮影します。(今回はpng画像で撮影しました)

* 様々なパターン画像までの距離や傾ける角度で撮影する
* 画像の端までパターンの内部の交点が来ている画像を撮影する

以下に撮影例を示します。

![](/images/rpi-fisheye-image-1.png =320x)
![](/images/rpi-fisheye-image-2.png =320x)
![](/images/rpi-fisheye-image-3.png =320x)

## 画像から校正用データを生成

以下のプログラムを実行します。普通のカメラの歪みの校正データ用の関数ではなく、魚眼レンズ用の`cv2.fisheye.calibrate()`を呼び出しています。

```python
import cv2
import numpy as np
import glob

# チェスボードのサイズ (内部コーナーの数)
CHECKERBOARD = (10, 7)  # 例: 横に6個、縦に9個の内部コーナー

# チェスボードの各マスのサイズ (mm)
SQUARE_SIZE = 25.0  # 例: 25mm

# 画像のパス
image_dir = "./"  # 撮影したチェスボード画像のディレクトリ
images = glob.glob(image_dir + "*.png")  # または "*.jpg" など

# 物体座標と画像座標を格納するリスト
objpoints = []  # 3D空間内のチェスボードコーナーの座標
imgpoints = []  # 画像内のチェスボードコーナーのピクセル座標

# チェスボードの物体座標を生成
objp = np.zeros((1, CHECKERBOARD[0] * CHECKERBOARD[1], 3), np.float32)
objp[0, :, :2] = np.mgrid[0:CHECKERBOARD[0], 0:CHECKERBOARD[1]].T.reshape(-1, 2)
objp *= SQUARE_SIZE  # マスのサイズを考慮

# 各画像に対して処理
for fname in images:
    img = cv2.imread(fname)
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    h, w = gray.shape[:2]

    # チェスボードのコーナーを検出
    ret, corners = cv2.findChessboardCorners(gray, CHECKERBOARD, None)

    if ret:
        objpoints.append(objp)

        # コーナーの精度を向上させる
        criteria = (cv2.TERM_CRITERIA_EPS + cv2.TERM_CRITERIA_MAX_ITER, 30, 0.1)
        corners2 = cv2.cornerSubPix(gray, corners, (11, 11), (-1, -1), criteria)
        imgpoints.append(corners2)

        # コーナーを描画 (デバッグ用)
        cv2.drawChessboardCorners(img, CHECKERBOARD, corners2, ret)
        cv2.imshow("img", img)
        cv2.waitKey(1)  # 少し待つ

cv2.destroyAllWindows()

# 魚眼レンズのキャリブレーション
K = np.eye(3)  # 初期カメラ行列
D = np.zeros((4, 1))  # 初期歪み係数
rvecs = [np.zeros((1, 1, 3), dtype=np.float64) for i in range(len(objpoints))]
tvecs = [np.zeros((1, 1, 3), dtype=np.float64) for i in range(len(objpoints))]

rms, K, D, rvecs, tvecs = cv2.fisheye.calibrate(
    objpoints,
    imgpoints,
    (w, h),
    K,
    D,
    rvecs,
    tvecs,
    flags=cv2.fisheye.CALIB_RECOMPUTE_EXTRINSIC + cv2.fisheye.CALIB_CHECK_COND + cv2.fisheye.CALIB_FIX_SKEW,
    criteria=(cv2.TERM_CRITERIA_EPS + cv2.TERM_CRITERIA_MAX_ITER, 30, 1e-6)
)

print("RMS:", rms)
print("K:\n", K)
print("D:\n", D)

# キャリブレーションデータを保存 (オプション)
np.savez("fisheye_calibration.npz", K=K, D=D, DIM=(w, h))
print("Calibration data saved to fisheye_calibration.npz")
```

## 校正用データで画像を補正

```python
import cv2
import numpy as np
import sys
def undistort_fisheye_image(input_image_path, output_image_path, K, D, DIM):
    """
    魚眼レンズ画像を指定されたパラメータで補正します。

    Args:
        input_image_path (str): 歪んだ魚眼レンズ画像のパス
        output_image_path (str): 補正後の画像の保存パス
        K (numpy.ndarray): カメラ行列 (3x3)
        D (numpy.ndarray): 歪み係数 (4x1 または 8x1)
        DIM (tuple): 歪んだ画像の寸法 (幅, 高さ)
    """

    img = cv2.imread(input_image_path)
    if img is None:
        print(f"Error: Could not read image from {input_image_path}")
        return

    # オプション: より詳細なマッピングのための新しいカメラ行列の計算
    # balance=1.0 で画像の全体を表示、balance=0.0 で中心部分のみを表示
    new_K = cv2.fisheye.estimateNewCameraMatrixForUndistortRectify(K, D, DIM, np.eye(3), balance=0.0)
    map1, map2 = cv2.fisheye.initUndistortRectifyMap(K, D, np.eye(3), new_K, DIM, cv2.CV_16SC2)
    undistorted_img = cv2.remap(img, map1, map2, interpolation=cv2.INTER_LINEAR, borderMode=cv2.BORDER_CONSTANT)

    cv2.imwrite(output_image_path, undistorted_img)
    print(f"Undistorted image saved to {output_image_path}")

# キャリブレーションデータの読み込み
calibration_data = np.load("fisheye_calibration.npz")  # キャリブレーションデータを保存したファイル名
K = calibration_data["K"]
D = calibration_data["D"]
DIM = calibration_data["DIM"]

args = sys.argv
if len(args) != 2:
    print("Usage: python undistort_image.py <input_image_path>")
    sys.exit(1)

input_image_path = args[1]
output_image_path = input_image_path.replace(".png", "_undistorted.png")

undistort_fisheye_image(input_image_path, output_image_path, K, D, DIM)

# 補足: 画像の表示
# 補正結果をすぐに確認したい場合は、以下のコードを追加できます。
# undistorted_img = cv2.imread(output_image_path)
# cv2.imshow("Undistorted Image", undistorted_img)
# cv2.waitKey(0)
# cv2.destroyAllWindows()
```

以下のように補正されます。

![](/images/rpi-fisheye-image-1-undistorted.png =320x)
![](/images/rpi-fisheye-image-2-undistorted.png =320x)
![](/images/rpi-fisheye-image-3-undistorted.png =320x)
