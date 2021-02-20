# 荔枝FM

## 1、【算法面试】n个数，找出其中最小的k个数，写出代码，要求最坏情况下的时间复杂度不能高于O(nlogk)



- 各种排序算法的实现和复杂度,稳定性二叉树的前中后序遍历
- 翻转句子中单词的顺序
- 用栈模拟队列(或用队列模拟栈)
- 对10亿个数进行排序,限制内存为1G
- 采用分治的思路
- 去掉(或找出)两个数组中重复的数
- 排序和hash两种思路
- 将一颗二叉树转换成其镜像
- 确定一个字符串中的括号是否匹配
- 给定一个开始词,一个结束词,一个字典,如何找到从开始词到结束词的最短单词接龙
- 考虑使用深度优先算法
- 如何查找两个二叉树的最近公共祖先
- 递归与非递归皆可实现

2、一个json字符串  是否正确匹配

3、数据量非常大的情况下   采用什么方式进行幂等校验。

4、Hystrix的服务熔断和服务降级


    //如有自定义的老师头像 优先显示
            List<GetTeacherPhotoRespDTO> teacherPhotoDTOS = teacherService.getTeacherPhoto(teacherList.stream().map(x -> x.getId()).collect(Collectors.toList()));
            if(CollUtil.isNotEmpty(teacherPhotoDTOS)) {
                Map<String, GetTeacherPhotoRespDTO> teacherPhotoRespDTOMap = teacherPhotoDTOS.stream()
                        .collect(Collectors.toMap(x -> x.getTeacherId(), Function.identity(), (n, o) -> o));
                teacherList.forEach(x ->  x.setPicUrl(Optional.ofNullable(teacherPhotoRespDTOMap.get(x.getId()))
                        .filter(t -> StringUtils.isNotBlank(t.getTeacherPhotoUrl()))
                        .map(t -> t.getTeacherPhotoUrl())
                        .orElse(x.getPicUrl())));
            }