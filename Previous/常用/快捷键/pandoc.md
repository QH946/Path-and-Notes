#### pdf转word

使用PDFXEdit

#### word转md

切换到word所在目录

pandoc -f docx -t markdown --extract-media ./ -o svn.md 11-SVN_k.docx
最后一个参数为word文件名称，倒数第二个参数为生成的md文件名称 ./为导出图片存放的位置

