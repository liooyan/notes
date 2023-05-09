![FST-结构](FST-结构.svg)







![FST-实例化图一](FST-实例化图一.svg)















![FST-实例化图二](FST-实例化图二.svg)



![FST-实例化图三](FST-实例化图三.svg)



![FST-实例化图四](FST-实例化图四.svg)









flags 含义

- BIT_FINAL_ARC： 节点对应字符是否是最后一个节点
- BIT_LAST_ARC ：节点对应字符是否是当前节点最后一个出度
- BIT_TARGET_NEXT： 下一个字符区间是是否是当前字符的下一个字符
- BIT_STOP_NODE： 当前节点是不是个终止符
- BIT_ARC_HAS_OUTPUT ： 是否有output值
- BIT_ARC_HAS_FINAL_OUTPUT ：是否有final output值
