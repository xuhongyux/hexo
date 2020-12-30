---
title: Lambda的实际运用
date: 2019-11-12 16:00:20
categories: Java 集训 
---

## 过滤

### 实体类属性为条件

```java
List<Student> lists = new ArrayList<>();
        Student student = new Student();
        student.setName("laoli");
        student.setNumber(1);
        lists.add(student);
        Student student1 = new Student();
        student1.setName("erzi");
        student1.setNumber(2);
        lists.add(student1);
        Student student3 = new Student();
        student3.setName("three");
        student3.setNumber(1);
        lists.add(student3);
        List<Student> collect = lists.stream().filter(s -> s.getNumber()==1&& !s.getName().equals("laoli")).collect(Collectors.toList());
        for (Student student2 : collect) {
            System.out.println(student2.getName());
        }
```

结果 thre

关键代码

```java
List<Student> collect = lists.stream().filter(s -> s.getNumber()==1&& !s.getName().equals("laoli")).collect(Collectors.toList());
```

### List转Map 

#### 收集实体类中的一个属性，以ID为key

```java
Map<Integer,String> idToName = people().collect(
			    Collectors.toMap(Person::getId, Person::getName));
```

#### 收集整个实体类，以ID为key

```java
java.util.function.Function是函数式接口，它的特点是有且只有一个抽象方法。
对于函数式接口，形如这样的表达式（x -> x + 1）能够实例化一个对象。
并且该表达式正是那个唯一的抽象方法的实现。
Map<Integer,String> idToName = people().collect(
			    Collectors.toMap(Person::getId, po -> po));
```

如果多个元素映射同一个键，那么就会存在冲突，为了防止冲突，一般情况如下

```java
List<ExamDownloadPo> examDownloads = examDownloadCacheService.selectByExample(example);
    	Map<String, ExamDownloadPo> examDownloadMap = examDownloads.stream().collect(Collectors.toMap(po -> po.getObjectId(), po -> po, (po1, po2) -> po1));
    	
```

### List<User> 获取其中一个属性

```java
List<String> studIds = studentDtos.stream().map(po -> po.getId()).collect(Collectors.toList());
```



### 滤重

distinct（）

使用hashCode（）和equals（）方法来获取不同的元素。因此，我们的类必须实现hashCode（）和equals（）方法。如果distinct（）正在处理有序流，那么对于重复元素，将保留以遭遇顺序首先出现的元素，并且以这种方式选择不同元素是稳定的。在无序流的情况下，不同元素的选择不一定是稳定的，是可以改变的。distinct（）执行有状态的中间操作。在有序流的并行流的情况下，保持distinct（）的稳定性是需要很高的代价的，因为它需要大量的缓冲开销。如果我们不需要保持遭遇顺序的一致性，那么我们应该可以使用通过BaseStream.unordered（）方法实现的无序流。

```java
List<String> list3 = list2.stream().distinct().collect(Collectors.toList());
```

filter()过滤

```java
List<ExamStatisticsSchoolVo> examStatisticsSchoolVoss = examStatisticsSchoolVos.stream().filter(distinctByKey(ExamStatisticsSchoolVo::getExamId)).collect(Collectors.toList());


private static <T> Predicate<T> distinctByKey(Function<? super T, ?> keyExtractor) {
        Map<Object,Boolean> seen = new ConcurrentHashMap<>();
        return t -> seen.putIfAbsent(keyExtractor.apply(t), Boolean.TRUE) == null;
    }
```

1. 根据某个实体属性去重
2. List<entity> list2 = list.stream().filter(distinctByKey((p) -> (p.get属性名称()))).collect(Collectors.toList());
3. 根据多个实体属性去重，我只是举了两个多个的话以此类推。
4. List<entity> list3 = list.stream().filter(distinctByKey((p) -> (p.get属性名称1()))).filter(distinctByKey((p) -> (p.get属性名称2()))).collect(Collectors.toList());

## 排序

###  实体类以一个属性为基础进行排序

[__sort()方法__](https://www.w3school.com.cn/js/jsref_sort.asp)

语法

```java
arrayObject.sort(sortby)  
 //sortby   可选。规定排序顺序。必须是函数。
```

* **返回值**
  对数组的引用。请注意，数组在原数组上进行排序，不生成副本。

* **说明**
  如果调用该方法时没有使用参数，将按字母顺序对数组中的元素进行排序，说得更精确点，是按照字符编码的顺序进行排序。要实现这一点，首先应把数组的元素都转换成字符串（如有必要），以便进行比较。

  如果想按照其他标准进行排序，就需要提供比较函数，该函数要比较两个值，然后返回一个用于说明这两个值的相对顺序的数字。比较函数应该具有两个参数 a 和 b，其返回值如下：

  若 a 小于 b，在排序后的数组中 a 应该出现在 b 之前，则返回一个小于 0 的值。
  若 a 等于 b，则返回 0。
  若 a 大于 b，则返回一个大于 0 的值。

```java
List<ExamStatisticsTeachRankingPo> examStatisticsRankings = examStatisticsTeachService.listExamStatisticsRankingByClass(
                classId,
                examId);
//排序
examStatisticsRankings.sort((o1, o2) -> {
	if (o1.getGradeRanking() > o2.getGradeRanking()) {
          return 1;
    } else if (o1.getGradeRanking() < o2.getGradeRanking()) {
            return -1;
            } else {
                return 0;
            }
    	 });
```

###  实体类以一个属性String排序

```java
// 排序班级名称
classes.sort((r1, r2) -> {
    String t1 = r1.getClassName();
    String t2 = r2.getClassName();
    return t1.compareTo(t2)>0  ? 1 : t1.compareTo(t2)<0 ? -1 : 0;
});
```

### 实体类以一个属性时间排序

```java
List<ImageCorrectRecordPo> collect = imageCorrectRecordPos.stream().sorted((po1, po2) -> {
            Date createTime1 = po1.getCreateTime();
            Date createTime2 = po2.getCreateTime();
            return createTime2.compareTo(createTime1);
        }).collect(Collectors.toList());
```

## 计算

### 求和

```java
double statisticalNum = bigDataAverageParamslist.stream().mapToDouble(BigDataAverageParams::getStatisticalNum).sum();
```



## 合并

### 实体类合并相同属性的数据

```java
//过滤 相同的班级 相同的时间，  将科目放在一起
List<TeacherInof> teacherInofForstOf = new ArrayList<>();
teacherInofForst.parallelStream().collect(Collectors.groupingBy(o -> ( o.getClassName() + o.getTime()),Collectors.toList())).forEach(
				(id, transfer)->{
					transfer.stream().reduce((a,b) -> new TeacherInof(a.getTime(),a.getClassName(),a.getRoleName(),a.getSubjectName()+","+b.getSubjectName(),a.getReason(),a.getGradeName())).ifPresent(teacherInofForstOf ::add);
				}
		);


@Data
@AllArgsConstructor
@NoArgsConstructor
public class TeacherInof {
    private Date time ;

    private String className;

    private String roleName;

    private String subjectName;

    private String reason;

    private String gradeName;
}


```

## 外传

### list转Map<String, List<>>

```java
	public static <T> Map<String, List<T>> listToKeyList(List<T> list, String... keys) {
		Map<String, List<T>> map = new LinkedHashMap<>();
		if (list == null || list.isEmpty() || keys == null || keys.length==0) {
			return map;
		}
		for (T obj : list) {
			if (obj == null) {
				continue;
			}
			String mapKey = getValue(obj, keys);
			if (!stringIsNullOrEmpty(mapKey)) {
				List<T> temp = map.get(mapKey);
				if (temp == null) {
					temp = new ArrayList<>();
					map.put(mapKey, temp);
				}
				temp.add(obj);
			}
		}
		return map;
	}
```

