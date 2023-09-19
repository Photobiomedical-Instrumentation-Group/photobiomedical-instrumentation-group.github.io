---
title: "Compresion of Facial Images"
date: "19-09-2023"
description: "A small study on lossless compression techniques for facial
images"
categories:
  - "Articles"
  - "Image compression"
  - "iPPG"
tags:
  - "Article"
  - "Image compression"
  - "iPPG"
menu: main # Optional, add page to a menu. Options: main, side, footer

lead: "A study on lossless compression techniques for images of the face" # Lead text
comments: false # Enable Disqus comments for specific page
authorbox: false # Enable authorbox for specific page
pager: true # Enable pager navigation (prev/next) for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
sidebar: false # Enable sidebar (on the right side) per page
---


**Relatório sobre uso de armazenamento e tempo no projeto de PA**

Um dos desafios deste projeto é o armazenamento do grande volume de
dados, especialmente imagens, necessário para treinamento da rede
neural. Neste documento apresento os experimentos que fiz para encontrar
a melhor maneira de armazenar estes dados, levando em conta tanto o
espaço ocupado pelos arquivos em disco quanto o tempo necessário para
leitura, escrita e processamento dos mesmos.

Todos estes experimentos foram feitos com uma mesma imagem fornecida
pelos nossos colegas do Japão, o primeiro frame do vídeo identificado
por
"AJ-11":![](Pictures/10000000000003C00000033477E8BC8171AA4816.png){width="4.3366in"
height="3.7043in"}

O principal software utilizado para todo o processamento de imagens, que
neste documento inclui principalmente leitura, gravação e compressão de
imagens, e conversões entre diferentes espaços de cor, foi a biblioteca
OpenCV (versão 3.4) para Python.

As especificações relevantes do computador utilizado nestes
experimentos, apelidado de *Honey Badger*, são: CPU Intel Core i7 de
quinta geração (6 cores, 12 threads, 3.3 Ghz, 15 MB de cache) e 4 pentes
de memória RAM Kingston (DDR-4, 2133 Mhz, 4 GB cada), totalizando 32 GB.
Os testes de velocidade de leitura e escrita de arquivos foram feitos em
um SSD Samsung de 512 GB (850 PRO 2.5\" SATA III 512GB).

**Separação da componente Hemoglobina**

** **Uma parte fundamental para este projeto é a obtenção da componente
Hemoglobina (doravante abreviada por Hem) através da análise de
componentes independentes (ICA, em inglês), pelo método de Tsumura et al
\[citar artigo\]. O resultado deste processamento no frame do vídeo
AJ-11 é a seguinte imagem:

![](Pictures/10000000000003C0000003341153BFCF3E1B41BA.png){width="3.9516in"
height="3.3752in"}

Para atingir a precisão numérica requerida por este projeto, é
necessário que estas imagens sejam armazenadas com precisão dupla, ou
seja, 16 bits por canal (48 bits por pixel) ao invés dos 8 bits por
canal (24 bits por pixel) usuais para imagens RGB. A imagem da
componente Hem, portanto, ocupa 2 vezes o espaço de uma imagem
não-processada quando ambas são armazenadas sem compressão.

Como a imagem da componente Hem é o resultado do processamento de uma
imagem RGB através de um algoritmo determinístico, seria teoricamente
possível armazenar somente as imagens RGB e reprocessá-la toda vez que a
componente Hem for usada. Para verificar a praticidade deste método,
reprocessamos a mesma imagem RGB 100 vezes para obter o tempo médio
levado por este algoritmo: 0,34 ± 0,01 s.

Esperamos obter gravações do rosto de voluntários de 2,5 min a 160 FPS,
o que corresponde a 24000 frames por vídeo. Se leva 0,34 segundos para
processar cada frame, o vídeo inteiro leva 2,27 horas. Esperar este
tempo para reprocessar cada vídeo a cada vez que precisarmos usar a
componente Hem é impraticável, principalmente considerando o caráter
exploratório deste projeto, em que buscamos processar estes vídeos de
diferentes maneiras e usá-los para o treinamento de uma rede neural.
Faz-se necessário, portanto, armazenar estas imagens pré-processadas em
disco.

\[1\] Independent Component Analysis of Skin Color Image

**Compressão das imagens**

O formato PNG pode armazenar matrizes de pixels com diferentes níveis de
compressão sem perdas. Quando usamos as funções imwrite \[1\] e imread
\[2\] do OpenCV para escrita e leitura de imagens, respectivamente, a
compressão e descompressão das imagens é feita por intermédio do codec
libpng \[1, 2\], que por sua vez emprega os algoritmos de compressão da
biblioteca zlib \[3\].

Por meio da zlib, o OpenCV suporta diferentes algoritmos de compressão
para imagens PNG, e 10 níveis de compressão para cada algoritmo, onde o
nível 0 representa nenhuma compressão e o 9, a máxima. Existe um
trade-off entre tempo de processamento e tamanho final do arquivo para
cada nível e algoritmo de compressão: alguns algoritmos são mais
adequados para tipos específicos de imagens. A documentação do OpenCV
lista os algoritmos disponíveis (na documentação, "estratégias"), e dá
uma noção da aplicabilidade de cada um \[4\].\
\
É importante ressaltar que todas as estratégias e níveis de compressão
para o formato PNG são lossless, ou seja, sem perda de informação.
Outros formatos, como o JPEG, suporta compressão com perdas, ou "lossy",
que tipicamente produzem imagens que ocupam muito menos espaço em disco
e com tempo de processamento significativamente menor. Para a aplicação
pretendida para nossas imagens -- análise de sinais biomédicos -- é
fundamental usar compressão sem perdas, uma vez que a compressão com
perdas pode ofuscar, eliminar ou mesmo adulterar o sinal de interesse.

**Níveis de compressão para a estratégia Default**

A Figura 1 apresenta gráficos do tempo de processamento, escrita,
leitura e tamanho ocupado em disco da imagem comprimida, para cada nível
de compressão. Para tal utilizamos a imagem RGB na introdução deste
documento.

![](Pictures/10000201000003200000032A2324B6374A79DB91.png){width="4.1181in"
height="4.1693in"}

Figura 1: Tempos de escrita, leitura e tamanho do arquivo para uma
imagem RGB de rosto, usando a estratégia de compressão "default" do
OpenCV. O tempo de escrita inclui o processamento necessário para
compressão, e o de leitura, para descompressão. O eixo à esquerda mostra
o fator multiplicativo de aumento ou redução do parâmetro em questão, em
relação à imagem não comprimida. Cada ponto corresponde ao valor médio
de 30 repetições de cada processo.

A Figura 2 apresenta os mesmos gráficos, mas para a imagem da componente
Hem.

Figura 2: Tempos de escrita, leitura e tamanho do arquivo para uma
imagem Hem de rosto, usando a estratégia de compressão "default" do
OpenCV. O tempo de escrita inclui o processamento necessário para
compressão, e o de leitura, para descompressão. O eixo à esquerda mostra
o fator multiplicativo de aumento ou redução do parâmetro em questão, em
relação à imagem não comprimida. Cada ponto corresponde ao valor médio
de 30 repetições de cada
processo.![](Pictures/10000201000003200000032A9B497438507577FC.png){width="4.1075in"
height="4.1591in"}

Em ambos os casos, o nível 9 apresentou menor tamanho do arquivo e tempo
de leitura, e o maior tempo de escrita. Um tempo de escrita maior é mais
aceitável em relação ao tempo de leitura e tamanho do arquivo, que são
parâmetros mais importantes -- cada imagem pode ser comprimida somente
uma vez, mas espera-se que seja lida inúmeras vezes durante a análise de
dados e treinamento da rede neural. O tamanho do arquivo é outro
parâmetro importante, não só porque precisaremos de uma quantidade
grande de imagens, como também o menor tamanho torna a imagem mais
rapidamente transferida e gravada.

**Estratégias de compressão com o nível 9**

Tendo estabelecido o nível de compressão 9 como o mais adequado,
testamos os algoritmos de compressão disponíveis no OpenCV (na
documentação, "estratégias" de compressão) nas imagens RGB e Hem, em
todos os casos adotando o nível 9;

![](Pictures/10000201000003200000032A563931E641C4F756.png){width="4.561in"
height="4.6181in"}

A Figura 3 apresenta o efeito de cada algoritmo de compressão na imagem
RGB.

Figura 3: Tempos de escrita, leitura e tamanho do arquivo para uma
imagem RGB de rosto, comprimida usando cada algoritmo de compressão
disponível no OpenCV, e adotando nível de compressão 9. O tempo de
escrita inclui o processamento necessário para compressão, e o de
leitura, para descompressão. O eixo à esquerda mostra o fator
multiplicativo de aumento ou redução do parâmetro em questão, em relação
à imagem não comprimida. Cada ponto corresponde ao valor médio de 30
repetições de cada processo.

A Figura 4 apresenta os resultados da mesma análise, mas com a imagem
Hem.![](Pictures/10000201000003200000032A3CD305A7A5068D2F.png){width="4.7008in"
height="4.7598in"}

Figura 4: Tempos de escrita, leitura e tamanho do arquivo para uma
imagem Hem de rosto, comprimida usando cada algoritmo de compressão
disponível no OpenCV, e adotando nível de compressão 9. O tempo de
escrita inclui o processamento necessário para compressão, e o de
leitura, para descompressão. O eixo à esquerda mostra o fator
multiplicativo de aumento ou redução do parâmetro em questão, em relação
à imagem não comprimida. Cada ponto corresponde ao valor médio de 30
repetições de cada processo.

Nota-se que a compressão para a imagem Hem é menos eficaz do que a da
imagem RGB, em termos do fator de redução do tamanho do arquivo. Para
ambas as imagens, observa-se que que todos os algoritmos de compressão
apresentaram tempos de leitura e tamanho do arquivo final semelhantes
(exceto para a estratégia *fixed, *que resultou em tamanhos de arquivos
notadamente maiores).

As estratégias *huffman_only* e *rle* resultaram em tempos de escrita
muito menores que as demais para ambos os tipos de imagens. Entre essas
duas, a *rle* mostrou-se ligeiramente melhor que a huffman_only em todos
os parâmetros para a imagem Hem. Adotamos, portanto, o algoritmo rle
para todas as imagens deste projeto, a fim de simplificar o código e as
nossas estimativas de armazenamento necessário.

**Conversões entre modelos de cor**

Avaliamos a possibilidade de armazenar uma mesma imagem em diferentes
modelos de cor, uma vez que este projeto prevê múltiplas conversões das
imagens entre os modelos CIE L\*A\*B\*, CIE L\*u\*v\*, XYZ, HSV, HSL e
RGB (o modelo original), a fim de auxiliar no processamento de vídeo e
treinamento das redes neurais.

Para trabalhar com imagens em diferentes modelos de cor, é importante
que elas estejam armazenadas com alto nível de precisão numérica, neste
caso, 32 bits por canal. Isso impossibilita a gravação das imagens no
formato PNG, uma vez que este é limitado a, no máximo, 16 bits por
canal. Nestes testes, gravamos cada imagem no formato EXR (implementado
no OpenCV por meio da biblioteca OpenEXR \[6\]).

O formato EXR apresenta algoritmos de compressão distintos do PNG, então
antes de avaliarmos o tamanho dos arquivos gerados para cada modelo de
cor, testamos cada algoritmo de compressão disponível no OpenCV para
este formato \[7\]. A Figura 5 apresenta os resultados obtidos.

Figura 5: Tempos de escrita, leitura e tamanho do arquivo para uma
imagem RGB de rosto, comprimida usando cada algoritmo de compressão
disponível no OpenCV para o formato EXR. O tempo de escrita inclui o
processamento necessário para compressão, e o de leitura, para
descompressão. O eixo à esquerda mostra o fator multiplicativo de
aumento ou redução do parâmetro em questão, em relação à imagem não
comprimida. Cada ponto corresponde ao valor médio de 30 repetições de
cada
processo.![](Pictures/10000201000003200000032A6B92C100AFA7FC1E.png){width="4.639in"
height="4.6965in"}

Pela Figura 5 verifica-se que o algoritmo PIZ apresentou o menor tamanho
do arquivo, um dos menores tempos de escrita e tempo de leitura
razoável. Logo, o adotamos para os testes subsequentes.

Adotando o algoritmo de compressão PIZ, realizamos testes a fim de
verificar se há um ganho na velocidade de acesso das imagens ao
gravá-las em disco em cada um destes modelos, ao invés de convertê-las
tantas vezes quanto for necessário processá-las nestes diferentes
modelos de cor. Em particular, comparamos o tempo de conversão entre o
modelo RGB e os demais com a velocidade de leitura da imagem já
convertida para o modelo de interesse. Os resultados estão resumidos na
Figura 6.

![](Pictures/10000201000003200000032ABC56FF3DC3EB6DA4.png){width="4.811in"
height="4.8717in"}

Figura 6: Tempos de conversão, escrita, leitura e tamanho do arquivo
para uma imagem RGB de rosto, para 5 modelos de cor, e adotando o
algoritmo de compressão PIZ. O tempo de escrita inclui o processamento
necessário para compressão, e o de leitura, para descompressão. O eixo à
esquerda mostra o fator multiplicativo de aumento ou redução do
parâmetro em questão, em relação à imagem não comprimida. Cada ponto
corresponde ao valor médio de 30 repetições de cada processo.

A informação crucial da Figura 6 é que o tempos de conversão do modelo
RGB para cada um dos 5 modelos avaliados são da ordem de $$10⁻⁵$$ a
$$10⁻⁴$$ segundos, enquanto que os tempos de leitura estão na ordem de
$$10⁻²$$ segundos. O armazenamento de cada uma destas imagens ocuparia,
ainda, mais que 8 vezes o espaço ocupado por uma imagem RGB no formato
PNG. Verifica-se, portanto, que não é prático nem desejável (em termos
de possíveis ganhos em velocidade de acesso) armazenar as imagens em
diferentes modelos de cor ao invés de convertê-las cada vez que for
necessário.

**Estimativa do espaço de armazenamento requerido**

Verificamos que a estratégia de compressão *rle* e o nível de compressão
9 forneceram os melhores resultados em termos da redução no tamanho do
arquivo e tempos de leitura e escrita para o formato PNG, para o tipo de
imagens com a qual lidamos. Adotando estes parâmetros, podemos calcular
o espaço necessário para armazenar um único frame de vídeo, contando com
as imagens RGB e Hem:

$${S_{\mathit{frame}} = {S_{\mathit{RGB}} + S_{\mathit{Hem}}} = 0,675}{\mathit{MB} + 3,853}{\mathit{MB} = 4,528}\mathit{MB}$$

Cada medida será um vídeo de 2,5 min a 160 FPS. Logo, o espaço demandado
por cada medida será:

$${S_{\mathit{medida}} = \frac{({\mathit{duração}\mathit{do}\mathit{vídeo}})}{(\mathit{FPS})}}{S_{\mathit{frame}} = N_{\mathit{frames}}}{S_{\mathit{frame}} = 24000}{S_{\mathit{frame}} = 108,7}\mathit{GB}$$

O espaço ocupado pelas 30 medidas que já temos é de 2,7 TB. Este número
é menor que os 3,3 TB estimados pelo cálculo de *S*~*medida\ *~acima.
Isso se deve à análise da performance dos algoritmos de compressão ter
se baseado no frame mais pesado que encontramos em nosso conjunto de
dados, justamente para garantir que nossas estimativas do espaço
necessário sejam suficientes na prática.

O espaço total demandado para armazenar medidas de 40 novos voluntários,
com 4 vídeos por voluntário, além das 30 medidas que já temos é,
portanto:

$${S_{\mathit{total}} = {{N_{\mathit{voluntarios}}N_{\mathit{medidas}}S_{\mathit{medida}}}_{❑} + 3,3}}{\mathit{TB} = 160}{S_{\mathit{medida}} + 3,3}{\mathit{TB} = 20,7}\mathit{TB}$$

Armazenar estes dados sem qualquer forma de backup condicionaria o
sucesso do projeto a eventualidades comuns, como a falha em discos
rígidos, corrupção de dados, deleção acidental, entre outras. Faz-se
necessário, portanto, adotar algum sistema de redundância no
armazenamento dos vídeos. Um sistema RAID pode cumprir este propósito.

O sistema RAID-5 é uma maneira de utilizar uma porção do armazenamento
disponível em múltiplos discos a bits de paridade, de modo a
proporcionar tolerância a falhas a um custo reduzido de espaço em disco.
Soluções de NAS (Network Attached Storage) costumam ter 4 baías para
discos rígidos. O espaço efetivamente disponível para armazenamento de
dados seria 75% do total, e os 25% seriam dedicados à proteção contra
falhas. Logo, o espaço demandado, contando com o requerido pelo RAID-5,
é de 27,6 TB.

É necessário para a implementação do RAID-5 que todos os discos possuam
a mesma capacidade de armazenamento. Discos rígidos costumam ter uma
capacidade em TB múltipla de 2, portanto, conviria adotar 4 discos de 8
TB cada, fornecendo um armazenamento disponível de 24 TB e 8 TB de
redundância. Esta configuração forneceria o armazenamento necessário,
com 3,3 TB extras para implementar novas ideias ao longo do projeto. O
algoritmo do RAID-5 é tal que um destes 4 discos poderia falhar
completamente sem que haja qualquer perda de dados, e, como os dados são
distribuídos entre os 4 discos, há ainda um ganho de 300% na velocidade
de leitura \[5\].

\[1\]
https://docs.opencv.org/3.4/d4/da8/group\_\_imgcodecs.html#ga288b8b3da0892bd651fce07b3bbd3a56

\[2\]
https://docs.opencv.org/3.4/d4/da8/group\_\_imgcodecs.html#ga288b8b3da0892bd651fce07b3bbd3a56

\[3\]
[*http://www.libpng.org/pub/png/book/chapter09.html*](http://www.libpng.org/pub/png/book/chapter09.html)

\[4\]
[*https://docs.opencv.org/4.x/d8/d6a/group\_\_imgcodecs\_\_flags.html#gaa60044d347ffd187161b5ec9ea2ef2f9*](https://docs.opencv.org/4.x/d8/d6a/group__imgcodecs__flags.html#gaa60044d347ffd187161b5ec9ea2ef2f9)

\[5\]
[*https://raid-calculator.com/default.aspx*](https://raid-calculator.com/default.aspx)

\[6\] [*https://openexr.com/en/latest/*](https://openexr.com/en/latest/)

\[7\]
[*https://docs.opencv.org/4.7.0/d8/d6a/group\_\_imgcodecs\_\_flags.html#ga7682010f3485d86cd963504aa7ad6146*](https://docs.opencv.org/4.7.0/d8/d6a/group__imgcodecs__flags.html#ga7682010f3485d86cd963504aa7ad6146)
