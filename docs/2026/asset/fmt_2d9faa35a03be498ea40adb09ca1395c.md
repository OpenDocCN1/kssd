
注意在`CCssDocumentLayout::GetPage`的调用中，程序没有像发生异常那样调用`CLayoutBlock::BuildBlock`，而是调用了`Ptls5::FsUpdateBottomlessPage`，并且最终调用了`CTextBlock`的析构函数。在执行完析构函数之后，`CGenericElment`所在的`CTextBlock`的空间就释放了：

|  |  |
| --- | --- |
