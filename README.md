<p align="center">
  <img alt="Github top language" src="https://img.shields.io/github/languages/top/young2j/oxmltotext?color=56BEB8">
  <img alt="Github language count" src="https://img.shields.io/github/languages/count/young2j/oxmltotext?color=56BEB8">
  <img alt="Repository size" src="https://img.shields.io/github/repo-size/young2j/oxmltotext?color=56BEB8">
  <img alt="License" src="https://img.shields.io/github/license/young2j/oxmltotext?color=56BEB8">
<img alt="Github forks" src="https://img.shields.io/github/forks/young2j/oxmltotext?color=56BEB8" />
  <img alt="Github stars" src="https://img.shields.io/github/stars/young2j/oxmltotext?color=56BEB8" />
</p>


<p align="center">
  <a href="#dart-about">About</a>   |
  <a href="#sparkles-features">Features</a>   |  
  <a href="#white_check_mark-requirements">Requirements</a>   |  
  <a href="#hammer_and_wrench-installation">Installation</a>   |  
  <a href="#rocket-quick start">Quick Start</a>   |  
  <a href="#hammer-build tags">Build Tags</a>   |  
  <a href="#fire-benchmark">Benchmark</a>  
</p>



<br>

> 😡😡😡Dumping nuclear wastewater into the ocean, damn it! 💣🗾💥😤😤😤

# 🎯 About

Oxmltotext is a lightweight and efficient text content extractor mainly for OOXML files (typically referring to DOCX/XLSX/PPTX). Solutions are also available for PDF as well as DOC/XLS/PPT formats.

# ✨ Features

This repo provides the following functionalities:
- Extracting text content from DOCX/XLSX/PPTX format(files,readers or URL) , with the option to extract text from charts/diagrams by configuring settings. 
  It can also extract text from images within the files using default tesseract or custom OCR interfaces.
- Extracting text content from PDF format(files,readers or URL) using [`go-fitz`](https://github.com/gen2brain/go-fitz).
- Extracting text content from DOC format(files,readers or URL) using the [`antiword`](https://en.wikipedia.org/wiki/Antiword) command-line tool.
- Extracting text content from XLS format(files,readers or URL) using the [`xlstotext`](xlstotext/rs) program(compiled using rust).
- Extracting text content from PPT format(files,readers or URL) using the `tika server` (about tika, seehttps://tika.apache.org/).

⚠️ Please note that this repo does not validate the validity of each file format.

# ✅ Requirements

    golang >=1.21.0

# 🛠 Installation

```shell
go get -u github.com/young2j/oxmltotext@latest
```

# :rocket: Quick Start

## 1. Extract text from docx/xlsx/pptx format

For these formats, the interfaces are consistent. Taking docx as an example:

### plain text

```go
import (
	"fmt"

	"github.com/young2j/oxmltotext/docxtotext"
)

func main() {
	dp, err := docxtotext.Open("../filesamples/file-sample_100kb.docx")
	if err != nil {
		panic(err)
	}
	defer dp.Close() // Please remember to call the `Close` method to avoid memory leaks.

	texts, err := dp.ExtractTexts()
	if err != nil {
		panic(err)
	}

	fmt.Println(texts)
}
```

Output looks like this:

```
...
-------------------------------------------------------------------------------------
Comment for demo.
-------------------------------------------------------------------------------------
Page Header ForDemo
-------------------------------------------------------------------------------------
Page Foot ForDemo
-------------------------------------------------------------------------------------
Footnote for demo.
-------------------------------------------------------------------------------------
Endnote for demo.
```

### charts and diagrams

Extract text of charts and diagrams:

```go
func main() {
	dp, err := docxtotext.Open("../filesamples/file-sample_100kb.docx")
	if err != nil {
		panic(err)
	}
	defer dp.Close() // Please remember to call the `Close` method to avoid memory leaks.

	dp.SetParseCharts(true) // set true if you want to parse charts text
	dp.SetParseDiagrams(true) // set true if you want to parse diagrams text

	texts, err := dp.ExtractTexts()
	if err != nil {
		panic(err)
	}

	fmt.Println(texts)
}
```

Output looks like this:

```
...(other texts)
┌───────────────chart───────────────┐
 [系列 1]
 类别 1 类别 2 类别 3 类别 4 
 4.3 2.5 3.5 4.5 
 [系列 2]
 类别 1 类别 2 类别 3 类别 4 
 2.4 4.4000000000000004 1.8 2.8 
 [系列 3]
 类别 1 类别 2 类别 3 类别 4 
 2 2 3 5 
└───────────────────────────────────┘
┌──diagram──┐
 smartart 1 
 smartart 2 
 smartart3 
└───────────┘
...(other texts)
```

Of course, you can also remove the formatting borders through API settings.

### OCR

Extract text of images(OCR):

> If OCR interface is not set, default tesseract-ocr will be used. 
> So you should install tesseract-ocr first for different operation system.
>
> If you use apt as package manager, you can run:
>
> ```shell
> apt install -y --no-install-recommends libtesseract-dev # libs
> apt install -y --no-install-recommends tesseract-ocr-eng tesseract-ocr-chi-sim tesseract-ocr-script-hans # language packages
> ```
>
> If you use homebrew on MacOS, you can run:
>
> ```shell
> brew install tesseract
> brew install tesseract-lang # language packages
> ```
>
> For more details, see [tesseract](https://github.com/tesseract-ocr/tesseract)

```go
func main() {
	dp, err := docxtotext.Open("../filesamples/file-sample_100kb.docx")
	if err != nil {
		panic(err)
	}
	defer dp.Close() // Please remember to call the `Close` method to avoid memory leaks.
  
	dp.SetParseImages(true) // set true if you want to parse images text

	texts, err := dp.ExtractTexts()
	if err != nil {
		panic(err)
	}

	fmt.Println(texts)
}
```

Output looks like this:

```
...(other texts)
┌──────────────────────image──────────────────────┐
 姓名 韦小宝
 
 性 别 男 民族 汉
 
 出 生 1654 £12 2208
 
 ff 址 北京 市 东城 区 景山 前 街 4 号
 紫禁城 敬 事 房
 
 公民 身份 证 号 码 11204416541220243x
└─────────────────────────────────────────────────┘
...(other texts)
```

## 2. Extract text from pdf format

```go
import (
	"fmt"

	"github.com/young2j/oxmltotext/pdftotext"
)

func main() {
	pp, err := pdftotext.Open("../filesamples/file-sample_500kb.pdf")
	if err != nil {
		panic(err)
	}
	defer pp.Close() // Please remember to call the `Close` method to avoid memory leaks.
	
  // Extract the text of page 1,2
	// texts, err := pp.ExtractPageTexts(1,2) 
	texts, err := pp.ExtractTexts()
	if err != nil {
		panic(err)
	}

	fmt.Println(texts)
}
```

## 3. Extract text from doc format

> To work for a doc file, you need to install Antiword.
>
> ```shell
> apt install -y --no-install-recommends antiword
> # or on MacOS
> brew install antiword
> ```

```go
import (
	"fmt"

	"github.com/young2j/oxmltotext/doctotext"
)

func main() {
	texts, err := doctotext.ExtractFromPath("../filesamples/file-sample_100kb.doc")
	if err != nil {
		panic(err)
	}

	fmt.Println(texts)
}
```

## 4. Extract text from xls format

> To work for a xls file, you should first compile the `xlstotext` executable program using Cargo, and then add it to your environment variables.
>
> ```shell
> cd xlstotext/rs
> cargo build --relese
> # executable program: xlstotext/rs/target/release/xlstotext
> ```

```go
import (
	"fmt"

	"github.com/young2j/oxmltotext/xlstotext"
)

func main() {
	texts, err := xlstotext.ExtractFromPath("../filesamples/file-sample_100kb.xls")
	if err != nil {
		panic(err)
	}

	fmt.Println(texts)
}
```

## 5. Extract text from ppt format

> If you need to extract text from ppt files and the only solution you have is Apache Tika, then indeed, you would need to run a Tika server. For testing, you can run the follow command to start the server on your machine.
>
> ```shell
> # see tikaserver/local.sh
> wget --no-check-certificate https://dlcdn.apache.org/tika/2.9.1/tika-server-standard-2.9.1.jar
> java -jar tika-server-standard-2.9.1.jar
> ```
>
> Tika server runs on the default port 9998.

```go
import (
	"fmt"

	"github.com/young2j/oxmltotext/ppttotext"
)

func main() {
	texts, statusCode, err := ppttotext.ExtractFromPathByTika("../filesamples/file-sample_500kb.ppt", "http://localhost:9998/tika")
	if err != nil {
		panic(err)
	}
	fmt.Printf("tika server respose status code:%d\n", statusCode)
	fmt.Println(texts)
}
```

# :hammer: Build Tags

Due to the need to install additional dependencies and since it's not a frequent requirement, as well as the potential impact on performance, OCR (Optical Character Recognition) for image text is not enabled by default. This repo utilizes the Go build tag "ocr" for conditional compilation. If you want to enable the default OCR interface (unless you provide a custom OCR implementation), you need to add the "ocr" tag during program compilation.

Your build command should contain the tag named "ocr":

```shell
go build -tags ocr .
```

# 🔥 Benchmark

```shell
cd tikaserver
java -jar tika-server-standard-2.9.1.jar

cd ../benchmark
make bench_all
```

## bench_docx

```shell
goos: darwin
goarch: amd64
pkg: gobench
cpu: Intel(R) Core(TM) i5-1038NG7 CPU @ 2.00GHz
Benchmark_ParseDocxByGooxml-8                 31          36919814 ns/op         6993180 B/op     125551 allocs/op
Benchmark_ParseDocxByGodocx-8                 87          15920631 ns/op         3748453 B/op      83037 allocs/op
Benchmark_ParseDocxByDocconv-8               100          15065089 ns/op         6537887 B/op      76339 allocs/op
Benchmark_ParseDocxByOxmlToText-8            322           3338957 ns/op          830322 B/op      18999 allocs/op
Benchmark_ParseDocxByTika-8                    1        1407496211 ns/op          153448 B/op        270 allocs/op
PASS
ok      gobench 8.718s
```
## bench_xlsx

```shell
go test -benchmem -bench ^Benchmark_ParseXlsx -benchtime=1s 
goos: darwin
goarch: amd64
pkg: gobench
cpu: Intel(R) Core(TM) i5-1038NG7 CPU @ 2.00GHz
Benchmark_ParseXlsxByGooxml-8                102          11412378 ns/op         1875498 B/op      31877 allocs/op
Benchmark_ParseXlsxByTealeg-8                199           5918170 ns/op         1868823 B/op      34834 allocs/op
Benchmark_ParseXlsxByExcelize-8              151           7667531 ns/op         3001478 B/op      33366 allocs/op
Benchmark_ParseXlsxByOxmlToText-8            885           1243743 ns/op          651748 B/op       5642 allocs/op
Benchmark_ParseXlsxByCmd-8                    96          13075914 ns/op           64203 B/op         96 allocs/op
Benchmark_ParseXlsxByTika-8                   28          37159483 ns/op          122720 B/op         89 allocs/op
PASS
ok      gobench 13.312s
```

## bench_pptx

```shell
go test -benchmem -bench ^Benchmark_ParsePptx -benchtime=1s 
goos: darwin
goarch: amd64
pkg: gobench
cpu: Intel(R) Core(TM) i5-1038NG7 CPU @ 2.00GHz
Benchmark_ParsePptxByGooxml-8                 19          62656297 ns/op         9900626 B/op     228856 allocs/op
Benchmark_ParsePptxByDocconv-8                27          38691741 ns/op        38118799 B/op     159225 allocs/op
Benchmark_ParsePptxByOxmlToText-8            202           5872266 ns/op          962422 B/op      52602 allocs/op
Benchmark_ParsePptxByTika-8                   14          78999145 ns/op          117511 B/op         82 allocs/op
PASS
ok      gobench 9.278s
```

## bench_pdf

```shell
go test -benchmem -bench ^Benchmark_ParsePdf -benchtime=1s 
goos: darwin
goarch: amd64
pkg: gobench
cpu: Intel(R) Core(TM) i5-1038NG7 CPU @ 2.00GHz
Benchmark_ParsePdfByLpdf-8                    43          26011079 ns/op        16955363 B/op     157749 allocs/op
Benchmark_ParsePdfByDocconv-8                 30          38640756 ns/op          144589 B/op        237 allocs/op
Benchmark_ParsePdfByFitz-8                   115          10333881 ns/op         2571023 B/op         47 allocs/op
Benchmark_ParsePdfByTika-8                   100          10672199 ns/op         2571039 B/op         47 allocs/op
PASS
ok      gobench 8.797s
```
## bench_doc

```shell
go test -benchmem -bench ^Benchmark_ParseDoc[^x] -benchtime=1s 
goos: darwin
goarch: amd64
pkg: gobench
cpu: Intel(R) Core(TM) i5-1038NG7 CPU @ 2.00GHz
Benchmark_ParseDocByAntiword-8               178           5917250 ns/op           64217 B/op         96 allocs/op
Benchmark_ParseDocByTika-8                    74          14812343 ns/op          122649 B/op         95 allocs/op
PASS
ok      gobench 4.850s
```
## bench_xls

```shell
go test -benchmem -bench ^Benchmark_ParseXls[^x] -benchtime=1s 
goos: darwin
goarch: amd64
pkg: gobench
cpu: Intel(R) Core(TM) i5-1038NG7 CPU @ 2.00GHz
Benchmark_ParseXlsToText-8           133           8668585 ns/op           64110 B/op         96 allocs/op
Benchmark_ParseXlsByTika-8           196           5711302 ns/op          122623 B/op         85 allocs/op
PASS
ok      gobench 5.590s
```
## bench_ppt

```shell
go test -benchmem -bench ^Benchmark_ParsePpt[^x] -benchtime=1s 
goos: darwin
goarch: amd64
pkg: gobench
cpu: Intel(R) Core(TM) i5-1038NG7 CPU @ 2.00GHz
Benchmark_ParsePptByTika-8                     5         264227774 ns/op          117718 B/op         96 allocs/op
Benchmark_ParsePptByTikaCmd-8                  1        9126216268 ns/op          360560 B/op        132 allocs/op
Benchmark_ParsePptByUnoconv-8                  1        16115858505 ns/op         582416 B/op        112 allocs/op
PASS
ok      gobench 30.404s
```

## 📝 License

This project is under license from MIT. For more details, see the [LICENSE](LICENSE.md) file.

Made with ❤️ by  <a href="https://github.com/young2j" target="_blank">young2j </a>
