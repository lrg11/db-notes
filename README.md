# db-notes
Notes about database papers
# Float
## chimp
chimp主要是通过比tsxor更复杂的编码方式，对一些难以压缩，即不太可能有同样的浮点数在同一个时间窗口这种情况，压缩比要好于tsxor，此外，它还采用比特方式来存储，而不是按字节存储压缩后的数据，从bit而不是byte粒度来压缩数据，但是对于常见的烽火和理想数据集，表现不如tsxor，它维护一个128大小的原始数据窗口，并通过尾部相似性的索引数组来快速定位可作异或比较的数据。

通过各种实际时间序列数据集中浮点值的特性的研究发现，两个连续值之间的异或操作结果的尾部零的数量分布不太可能有大量尾部零。相反，大多数异或操作结果表现出相当数量的前导零。基于这些发现，产生了Chimp压缩算法，该算法基于前导零的数量分布，提供了一种非常空间高效的表示形式，连续值可以频繁地重用该表示。此外，当尾部零的数量足够多以实现节省时，该算法利用尾部零来实现卓越的压缩率。
Chimp算法利用多样化的时间序列展示的属性，在压缩和访问时间方面优于常见的流式方法（gorilla），并提供了更好的空间和速度之间的权衡。与较慢但极其有效的通用压缩方案相比（Snappy），在保持压缩和解压缩速度的同时，它提供了显著空间节省。
## tchimp
在tsxor基础上，引入了chimp的bit存储方式，通过对异或结果进行更细粒度的存储，进一步提升压缩比。
Elf
Elf可以通过其Eraser组件将浮点值转换为具有更多尾部零位的另一个值，在擦除步骤引入了一些精度损失，但这被恢复步骤所补偿，该步骤从被擦除的值中恢复原始值，保证无损属性的同时实现了显著的压缩比。利用浮点数据的特性，Eraser组件会擦除浮点值的最后几位，增加XOR结果中的尾部零位的数量，从而实现更高的压缩比，对XOR压缩编码方式也进行了优化。通过记录尾部零位的数量，并使用较少的位数存储非尾部位，ElfXORcmp可以使用比现有的基于XOR的压缩器更少的位数来压缩第一个值。

Elf相比于最佳竞争者Chimp128可以获得的效率提升取决于被压缩的数据集和浮点数十进制有效位数𝛽的值。根据论文中呈现的实验结果，如果𝛽不大，Elf压缩算法甚至比增强版Gorilla和Chimp的压缩比提高了8.7% ∼33.3%。特别是当𝛽=6时，Elf相对于Chimp128的压缩比增益最高（时间序列数据集和非时间序列数据集分别实现了33%和55%的相对改善）。然而，对于𝛽较大的数据集，ElfEraser无法增强包括Chimp128在内的基于异或的压缩器，因为对于较大的𝛽，ElfEraser放弃擦除以避免负增益。因此，Elf相对于Chimp128的效率提升取决于具体的数据集和𝛽的值。

## TSXOR
TSXor是一种针对时序数据的简单而有效的无损压缩算法。
TSXor通过窗口作为缓存，利用接近时间的值之间的冗余和相似性，实现了比Gorilla更好的压缩比。TSXor充分利用了缓存值的窗口，在85%的情况下，它使用单个字节来引用在滑动窗口中出现的相同的8字节值。而这对于使用基于范围的编码来压缩双精度浮点数值的Gorilla来说是不可能的。Gorilla的基于范围的编码对于具有较大范围的值在比特方面产生更高的开销，从而限制了其压缩效果。
关于TSXor的压缩速度，为了获得更好的压缩比和更快的解码速度而交换了压缩速度。在对每个要编码的值进行编码时，需要扫描整个窗口，这会降低压缩速度。与在压缩速度方面最快的Gorilla相比，TSXor的速度较慢。然而，TSXor实现了比Gorilla和FPC更好的压缩比和更快的解压缩速度，这对于减少训练时间而不影响准确性非常重要。改进TSXor的编码时间的方法，如利用矢量化指令对窗口内多个数据同时进行比较，或者利用类似chimp中的尾部相似性的的索引数组，以空间换取时间。
