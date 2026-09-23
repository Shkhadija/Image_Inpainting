# Image Inpainting — ResNet34-UNet

Maskalanmış şəkillərin bərpası (image inpainting). Model şəkildəki kvadrat boşluğu ətrafındakı kontekstə əsasən doldurur.

## Model

- **Arxitektura:** ImageNet-də öyrədilmiş ResNet34 encoder + U-Net tipli decoder (skip connection-larla)
- **Giriş:** RGB + maska (4 kanal), ölçü 128×128
- **Loss:** MSE + L1 + VGG16 perceptual loss + maskalanmış sahədə əlavə çəkili L1
- **Təlim:** Adam (lr=0.0005), `ReduceLROnPlateau`, early stopping; train zamanı təsadüfi 16–48 px kvadrat maskalar
- **Nəticə:** təlim epoch 56-da dayandı, ən yaxşı val loss 0.334

## Nəticələr

Val seti (2941 şəkil), sabit 32×32 maskalar. Hər iki model eyni qaydada qiymətləndirilib (kompozit: maskadan kənar piksellər orijinaldan götürülür).

| Metrik | Sadə U-Net | ResNet-UNet |
|---|---|---|
| PSNR (kompozit), dB | 32.41 | **35.17** |
| PSNR (maskalanmış sahə), dB | 20.37 | **23.13** |
| SSIM | 0.9640 | **0.9711** |
| LPIPS ↓ | 0.0542 | **0.0364** |

Kompozit PSNR şəklin yalnız 1/16 hissəsi modeldən gəldiyi üçün yüksək çıxır; modelin real bərpa keyfiyyətini maskalanmış sahə üzrə PSNR daha yaxşı əks etdirir.

## Dataset

[Image Inpainting (Random Objects Images Dataset)](https://www.kaggle.com/datasets/mohamedyabdelaziz/inpainting) — Kaggle

- `training/` qovluğundan 29 410 şəkil istifadə olunub (90% train / 10% val)
- License: Community Data License Agreement – Permissive, Version 1.0 (CDLA-Permissive-1.0)

## İşlətmə

Notebook Google Colab (GPU) üçün yazılıb. Dataset `.zip` kimi Google Drive-dan oxunur, notebook-dakı yolları öz Drive-ınıza görə dəyişin.

```
pip install torch torchvision scikit-image lpips matplotlib
```

## Məhdudiyyətlər

- Çıxışda yüngül bulanıqlıq var (MSE/L1 əsaslı loss-un tipik nəticəsi); adversarial (GAN) loss əlavə etmək kəskinliyi artıra bilər
- Model yalnız kvadrat maskalarla öyrədilib və yalnız 32×32 maskada qiymətləndirilib
- Ayrıca test seti yoxdur (val seti checkpoint seçimi üçün də istifadə olunub)
- Learning rate axtarışı loss-un ilkin versiyası ilə aparılıb
