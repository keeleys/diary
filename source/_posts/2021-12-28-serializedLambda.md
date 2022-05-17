---
title: 从lambda中获取Field
date: 2020-05-11 14:05:05
tags:
---

## 记录下自己封装的金蝶api操作类

### 金蝶查询分析
```json

// 这里是金蝶查询的 json对象，可以看到查询列需要自己一个个指定，返回的时候按查询的字段顺序返回一个二维数组
// 查询和返回都类似的用数组结构，没有用对象化抽象的话调用会写的很丑。要一个个字段的去写，去对应，然后自己实现转换
{
    "FieldKeys": "FID,FRECEIVEBILLENTRY_FENTRYID,FBILLTYPEID.FNAME,FBILLNO,FDATE,FSALEERID.FNAME,FSALEERID.FNUMBER,FRECACCOUNTNAME,FACCOUNTID.FNUMBER,FRECAMOUNTFOR,FPURPOSEID.FNUMBER,FBLEND,FDOCUMENTSTATUS,FREMARK,FCREATEDATE,FMODIFYDATE,FAPPROVEDATE,FSETTLETYPEID.FNUMBER,FSETTLETYPEID.FNAME,FWRITTENOFFAMOUNTFOR_D,FCONTACTUNIT.FNUMBER,FCONTACTUNIT.FName,FCOMMENT,FPAYUNIT.FName,FPAYUNIT.FNUMBER,FWRITTENOFFSTATUS,FTHIRDBILLNO,FRECAMOUNTFOR_E,F_ora_Assistant.FNUMBER",
    "FilterString": " FDOCUMENTSTATUS = 'C' and FBILLNO = 'abc' ",
    "FormId": "AR_RECEIVEBILL",
    "Limit": 20,
    "OrderString": "FCREATEDATE ASC",
    "StartRow": 0,
    "TopRowCount": null
}
```
### 添加查询条件类 ErpQueryCondition.java

>> 这个类是用来拼装查询条件的

```java
public class ErpQueryCondition<T> {
    public static final String AND = "and";
    public static final String OR = "or";
    private final List<String> queryList;

    public ErpQueryCondition() {
        queryList  = new ArrayList<>();
    }
    public ErpQueryCondition<T> or(Function<ErpQueryCondition<T>, ErpQueryCondition<T>> func){
        ErpQueryCondition<T> erpQueryCondition = new ErpQueryCondition<>();
        String subStr = func.apply(erpQueryCondition).getQueryString();
        queryList.add(OR + " ("+subStr+")");
        return this;
    }
    public ErpQueryCondition<T> and(Function<ErpQueryCondition<T>, ErpQueryCondition<T>> func){
        ErpQueryCondition<T> erpQueryCondition = new ErpQueryCondition<>();
        String subStr = func.apply(erpQueryCondition).getQueryString();
        queryList.add(AND + "("+subStr+")");
        return this;
    }

    public ErpQueryCondition<T> eq(boolean condition, SFunction<T,Object> func, Object value) {
        return this.operator(condition,"=",func,value);
    }
    public ErpQueryCondition<T> eq(SFunction<T,Object> func, Object val) {
        return eq(true, func,  val);
    }

    public ErpQueryCondition<T> lt(boolean condition, SFunction<T,Object> func, Object value) {
        return this.operator(condition,"<",func,value);
    }
    public ErpQueryCondition<T> lt(SFunction<T,Object> func, Object val) {
        return lt(true, func,  val);
    }
    public ErpQueryCondition<T> le(boolean condition, SFunction<T,Object> func, Object value) {
        return this.operator(condition,"<=",func,value);
    }
    public ErpQueryCondition<T> le(SFunction<T,Object> func, Object val) {
        return le(true, func,  val);
    }

    public ErpQueryCondition<T> gt(boolean condition, SFunction<T,Object> func, Object value) {
        return this.operator(condition,">",func,value);
    }
    public ErpQueryCondition<T> gt(SFunction<T,Object> func, Object val) {
        return gt(true, func,  val);
    }

    public ErpQueryCondition<T> ge(boolean condition, SFunction<T,Object> func, Object value) {
        return this.operator(condition,">=",func,value);
    }
    public ErpQueryCondition<T> ge(SFunction<T,Object> func, Object val) {
        return ge(true, func,  val);
    }

    public ErpQueryCondition<T> in(boolean condition, SFunction<T,Object> func, Collection<String> list) {
        if(condition && list!=null && !list.isEmpty()) {
            Collection<String> convertList = list.stream().map(it -> "\\'" + it.toString() + "\\'").collect(Collectors.toList());
            String listStr = "(" + StringUtils.join(convertList, ",") + ")";
            return this.operator(condition, "in", func, listStr);
        }
        return this;
    }
    public ErpQueryCondition<T> in(SFunction<T,Object> func, Collection<String> list) {
        return in(true, func,  list);
    }

    public ErpQueryCondition<T> ne(boolean condition, SFunction<T,Object> func, Object value) {
        return this.operator(condition,"!=",func,value);
    }
    public ErpQueryCondition<T> ne(SFunction<T,Object> func, Object val) {
        return ne(true, func,  val);
    }

    private ErpQueryCondition<T> operator(boolean condition, String optStr, SFunction<T,Object> func, Object value) {
        if(!condition){
            return this;
        }
        // 要处理的字符串,待优化成注册类型转行服务。
        String handlerStr;
        if(value instanceof Date){
            handlerStr = DateFormatUtils.format((Date)value, "yyyy-MM-dd HH:mm:ss");
        }else{
            handlerStr = value.toString();
        }
        String fieldName = SerializedUtil.getErpFiledValue(func);
        queryList.add(StrUtil.format("{} {} {} '{}' ",AND, fieldName,optStr,handlerStr));
        return this;
    }
    public String getQueryString(){
        String result =StrUtil.join("",queryList);
        // 去掉第一个 and
        if(StrUtil.isNotEmpty(result)){
            return StrUtil.subSuf(result,AND.length());
        }
        return result;
    }

}
```

### ErpConvert.java

> 这个类是用来转换查询和返回结果的

```java
public class ErpConvert<T> {

    public static final String STR_SPILT = ",";
    private List<String> erpQueryStrList;
    private Map<String,String> erpValueMap;
    private final Class<T> zClass;
    public ErpConvert(Class<T> zClass) {
        this.zClass = zClass;
        this.erpQueryStrList = new ArrayList<>();
    }
    public static <Z> ErpConvert <Z> newErp(Class<Z> classZ){
        return new ErpConvert<>(classZ);
    }

    public String getQueryString(){
        erpQueryStrList = new ArrayList<>();
        for(Field f: ReflectUtil.getFields(this.zClass)){
            this.handlerField(f, erpName->erpQueryStrList.add(erpName));
        }
        return String.join(STR_SPILT, erpQueryStrList);
    }

    public String getFormId(){
        ErpForm erpForm = this.zClass.getAnnotation(ErpForm.class);
        if(erpForm != null){
            return erpForm.value();
        }
        return null;
    }

    public ErpConvert<T> setResult(List<String> erpResultStrList){
        if(erpResultStrList.size() != erpQueryStrList.size()){
            throw new ErpException("查询和返回的字段数量不一致");
        }
        this.erpValueMap = new HashMap<>();
        for(int i = 0; i<erpResultStrList.size(); i++){
            erpValueMap.put(erpQueryStrList.get(i),erpResultStrList.get(i));
        }
        return this;
    }


    public ErpConvert<T> setResultErp(List<Object> list){
        List<String> strList =list.stream().map(it-> Optional.ofNullable(it).map(Object::toString).orElse("")).collect(Collectors.toList());
        return this.setResult(strList);
    }

    /**
     * 通过response返回的字符串 获取list返回结果
     * @param responseString
     * @return
     */
    public List<T> getList(String responseString){
        List<List<String>> receiveBillResult= JsonUtil.fromJson(responseString, new TypeReference<List<List<String>>>(){});
        return receiveBillResult.stream().map(it->this.setResult(it).getValue()).collect(Collectors.toList());
    }

    public T getValue(){
        T resultObj;
        try {
            resultObj = ReflectUtil.newInstance(this.zClass);
        } catch (Exception e) {
            throw new BusinessException(this.zClass.getName()+"反射对象失败");
        }
        if(erpValueMap.isEmpty()){
            throw new ErpException("清先设置erp的返回值");
        }
        for(Field f : ReflectUtil.getFields(this.zClass)){
            this.handlerField(f, erpName-> setFieldValue(resultObj,f,erpName));
        }
        return resultObj;
    }

    /**
     * 支持三种注解查询
     * @param field 要设置的字段
     * @param consumer 处理方法
     */
    private void handlerField(Field field, Consumer<String> consumer){
        ErpField erpField = field.getAnnotation(ErpField.class);
        if(erpField != null){
            consumer.accept(erpField.value());
            return;
        }
        JsonProperty jsonProperty = field.getAnnotation(JsonProperty.class);
        if(jsonProperty!=null){
            consumer.accept(jsonProperty.value());
            return;
        }
        JSONField jsonField = field.getAnnotation(JSONField.class);
        if(jsonField!=null){
            consumer.accept(jsonField.name());
        }
    }

    private void setFieldValue(Object object, Field f, String erpName){
        f.setAccessible(true);
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss");

        try {
           String value =  erpValueMap.get(erpName);
           if(value == null){
               f.set(object, null);
           }else if(f.getType().equals(Date.class)){
               f.set(object, StringUtil.isEmpty(value) ? null:sdf.parse(value));
           }else if(f.getType().equals(BigDecimal.class)){
               f.set(object, StringUtil.isEmpty(value) ? BigDecimal.ZERO:new BigDecimal(value));
           }else if(f.getType().equals(Long.class)){
               f.set(object, StringUtil.isEmpty(value) ? 0L:new BigDecimal(value).longValue());
           }else if(f.getType().equals(Integer.class)){
               f.set(object, StringUtil.isEmpty(value) ? 0:new BigDecimal(value).intValue());
           }else{
               f.set(object, value.contains(".0") ? MathUtils.subZeroAndDot(value):value);
           }
        } catch (IllegalAccessException | ParseException e) {
            throw new ErpException(f.getName()+"["+erpName+"]设置失败，"+e.getMessage());
        }
    }

}
```