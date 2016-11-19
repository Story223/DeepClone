# DeepClone
list数据缓存，需要考虑到引用问题，解决办法是使用clone

在开发android项目中使用到Adpater的常有的事
有时需要运用到list缓存原本的list数据，在完成一些操作后对比缓存的list数据，根据有变化的item的数据发送对应的请求
例如，编辑——》进行编辑操作，保存-》发送操作的Item的请求。
最暴力的是不进行，直接发送所有的数据，但这样会产生的流量，并且若用户频繁点击编辑保存编辑保存，同样会发送请求，显然是浪费流量

今天就运用到这样的缓存List，但是在使用起来却产生问题：
逻辑上的思路是，在初始化和更新list数据时创建或更新cachelist的数据，最后在发送请求前对比cachelist和adapter里面的list，从而发送有变化的item数据
  至于cachelist在activity创建还是在adapter里面创建，都没有影响，无非就是多写一个adapter.getCacheList()方法，
  
  所以我一开始写的代码是：
   private void cacheManagerList(){
        Log.e(TAG, "----cacheManagerList: managerBeanlist"+managerBeanlist.toString());
        if (!oldManagerBeanlist.isEmpty()){
            oldManagerBeanlist.clear();
        }
        for (ManagerGWBean bean :managerBeanlist){
            oldManagerBeanlist.add(bean);
        }
    }
    
    而对比方法是
    public static boolean isChangeManagerBean(ManagerGWBean newBean,ManagerGWBean oldBean){
            if (newBean.getJurisdiction()!=oldBean.getJurisdiction()){
                return true;
            }
            if (newBean.getIsshare()!=oldBean.getIsshare()){
                return true;
            }
            return false;
        }
        
    但是使用发现无论如何限制不使用cachelist，但最终发送请求前的cachelist都会变成跟list一样
    
    请教大神后得知是Java的赋值引用问题，无论是cachelist.add方法还是cachelist = list，cachelist的值都是list的引用，所以需要运用浅拷贝和深拷贝的知识。
    如果只是基本数据类型可以使用clone()方法
    例，
    ArrayList<String> list;
    ArrayList<String> list2 = list.clone();
    这比较简单
    但是如果像上述的例子的一样使用bean类的list，则需要用另一种办法，通过流的形式赋值
    在bean中加入一个方法
    例，
    public class ManagerGWBean implements Serializable{
    public Object deepClone() throws IOException,
            ClassNotFoundException {
        //将对象写到流里
        ByteArrayOutputStream bo = new ByteArrayOutputStream();
        ObjectOutputStream oo = new ObjectOutputStream(bo);
        oo.writeObject(this);
        //从流里读出来
        ByteArrayInputStream bi = new ByteArrayInputStream(bo.toByteArray());
        ObjectInputStream oi = new ObjectInputStream(bi);
        return (oi.readObject());
        }
    }
    注意：需要继承Serializable的接口
    
    而在cachelist赋值时，只要在调用这个方法就可以
    private void cacheManagerList() throws IOException, ClassNotFoundException {
        Log.e(TAG, "----cacheManagerList: managerBeanlist"+managerBeanlist.toString());
        if (!oldManagerBeanlist.isEmpty()){
            oldManagerBeanlist.clear();
        }
        for (ManagerGWBean bean :managerBeanlist){
            oldManagerBeanlist.add((ManagerGWBean) bean.deepClone());
        }
    }
    
    这样就能解决缓存list的使用问题。成功完成通过对比发送对应请求的功能。
    
    
    
