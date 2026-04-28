[npm i html2pdf.js@0.14.0](https://ekoopmans.github.io/html2pdf.js/)

html2pdf.js 使用 html2canvas 和 jsPDF 将任何网页或元素转换为完全在客户端的可打印 PDF。

```
import html2pdf from 'html2pdf.js'

const handleDownloadPdf = () => {
    const element = document.getElementById('previewQuotation')
    if (element) {
        nextTick(() => {
            html2pdf()
                .set({
                    margin: [10, 10, 10, 10], // 增加边距
                    filename: 'export.pdf',
                    html2canvas: {
                        scale: 2,
                        useCORS: true,
                        logging: false,
                        letterRendering: true,
                        allowTaint: true,
                        imageTimeout: 15000,
                    },
                    jsPDF: {
                        unit: 'mm',
                        format: 'a4',
                        orientation: 'portrait',
                    },
                    //@ts-ignore
                    pagebreak: {
                        mode: ['avoid-all', 'css', 'legacy'], // 避免在元素中间分页
                        // before: '.qr-code-info', // 在二维码区域前强制分页
                    },
                })
                .from(element)
                .save('报价单.pdf')
        })
    }
}
```