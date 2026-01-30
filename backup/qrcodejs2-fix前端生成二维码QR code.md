[npm install qrcodejs2-fix@0.0.1](https://www.npmjs.com/package/qrcodejs2-fix?activeTab=readme)


```
<div class="qr-code" ref="qrCodeRef"></div>

let qrCodeRef = ref()

import QRCode from 'qrcodejs2-fix'

const generateQrCode = (
    qrCodeUrl: string,
    filename: string = 'qrcode.png',
    width: number = 150,
    height: number = 150,
    t?: (key: string) => string
) => {
    // 先清除旧的二维码
    if (qrCodeRef.value) {
        qrCodeRef.value.innerHTML = ''
    }
    // 2. 使用 qrcodejs2-fix 生成二维码
    // QRCode 会在容器内创建一个 canvas 元素
    // correctLevel: 0 = L, 1 = M, 2 = Q, 3 = H
    new QRCode(qrCodeRef.value, {
        text: qrCodeUrl,
        width: width,
        height: height,
        colorDark: '#000000',
        colorLight: '#ffffff',
        correctLevel: 3, // H 级别，最高容错率
    })
}
```

