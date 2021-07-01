# filter
有Blog model和 Tag model,且blog和tag的关系是一对多。即一篇文章可以有多个标签


### 第一种情况： filter的字段都是主表上的字段
  生成sql语句时会生成相应的where语句
  要对blog的title和create_time进行筛选，可以有两种方式
  
  ```
  Blog.objects.filter(title='blog_title', create_time='2021-07-01') 
  Blog.objects.filter(title='blog_title').filter('create_time='2021-07-01')
  ```
  
  这两种方式是等效的，因为filter的字段都是主表字段，生成的sql都是
  
  ```
  select * from blog where title='blog_title' and create_time='2021-07-01')
  ```
  
  
### 第二种情况： filter的字段是关联表的字段
  这种情况下，每增加一个filter，生成sql语句时都会进行一次join并增加一个where条件
  
  要通过tag筛选blog,比如要筛选tag__name == "django" 
  ```
  Blog.objects.filter(tag__name='django')
  ```
  生成的sql为
  ```
  select blog.* form blog join blog_tags bl on bl.blog_id = blog.id join tag t on t.id=bl.tag_id
  where tag.name='django' 
  ```
  这是如果要增加一个筛选条件，tag颜色为红色,即tag name为django ,且color为red的blog
  
  ```
  Blog.objects.filter(tag__name='django', tag__color='red')
  ```
  此时生成的sql只是在where后面增加了一个color='red'的条件
   ```
  select blog.* form blog join blog_tags bl on bl.blog_id = blog.id join tag t on t.id=bl.tag_id
  where tag.name='django' and tag.color='red'
  ```
  
  但如果写成了
  ```
  Blog.objects.filter(tag__name='django').filter(tag__color='red')
  ```
  就会导致问题
  使用两个filter，生成的sql语句如下
  ```
   select blog.* form blog join blog_tags bl on bl.blog_id = blog.id join tag t on t.id=bl.tag_id
   join bl2 on bl2.blog_id=blog.id join tag t2 on t2.id=bl2.tag_id
  where t.name='django' and t2. color='red'
  ```
  如果说tag的name和color之间没有绑定关系，即name相同的标签，可能有红色的，可能有绿色的
  
  这时，假如一个blog，有两个标签，一个name为django， 颜色为绿色，一个name为mysql, 颜色为红色
  带入前面的sql中，第一个标签所在的表为t1, 第二个标签所在的表为t2, 
  tag1 name 为django , tag2 color为red, 会将这条blog筛选出来，但是它明显不符合我们本来的意图。
  
  所以如果是通过多个条件进行and筛选的话，只写一个filter,一定不会有问题。
  
  
  
  
  
  
  
  
 
