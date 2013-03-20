---
layout: post
category : lessons
tags : [SQLalchemy]
---
{% include JB/setup %}

自打接触了python，就被深深的吸引，首先是它频繁出现在我工作使用的著名GIS软件中，一次为了给一个GIS软件写点脚本，就尝试了一下，这一尝试不要紧，心从此就被他栓住了，再开发什么东西首先就考虑用python去做。

为什么会被python吸引的，我思考后得出这个结论：

1、语言简练，亲切，容易学习

2、开源的库众多，只要你能想到就能得到


得知有一个很NB的ORM框架SQLAlchemy，所以打算学习一下，从今天起就开始做学习笔记，希望能坚持下去，本笔记简要记录一些自己学习中的心得和要点，尽量多写一些代码范例供以后查看，学习前就说这么多了，下面就开始吧！

![](http://images.cnblogs.com/cnblogs_com/booolee/WindowsLiveWriter/SQLAlchemy_D760/image_thumb.png)

SQLAlchemy拥有多个独立的部分，可以单独或协同工作，下面是主要API组件： 

上图中最重要的两个接口是Object Relational Mapper（ORM）和SQL Expression Language

SQL Expression Language可以独立于ORM使用，但ORM需要使用SQL Expression Language构建。

要使用，必然要先安装，假定已经安装了python2.5并配置了路径，并下载了setuptools工具，然后可以使用命令:

	easy_install SQLalchemy

该命令从Python Cheese Shop下载SQLalchemy并安装到系统。

SQLalchemy需要特定数据库的DB-API，根据你要操作的数据库，可以选择不同的DB-API库，比如：oracle需要安装cx_oracle。

为了检查是否安装正确可以输入以下命令来检查已安装的sqlalchemy版本：

	>>>import sqlalchemy
	>>>sqlalchemy.__version__
	0.5.5

注意：练习均以内存SQLite数据库为对象

	>>> from sqlalchemy import create_engine
	>>> engine=create_engine('sqlite:///:memory:',echo=True) 

用create_engine建立一个数据库链接，eacho标志为True则会利用python标准logging模块输入产生的SQL过程。

	>>> from sqlalchemy import Table,Column,Integer,String,MetaData,ForeignKey
	>>> metadata=MetaData()
	>>> users_table=Table('users',metadata,
          	Column('id',Integer,primary_key=True),
          	Column('name',String),
          	Column('fullname',String),
          	Column('password',String)
          	) 

创建表的模式

	>>> metadata.create_all(engine)
	2009-08-26 16:06:02,812 INFO 	sqlalchemy.engine.base.Engine.0x...bbb0 PRAGMA table_info("users")
	2009-08-26 16:06:02,828 INFO sqlalchemy.engine.base.Engine.0x...bbb0 ()
	2009-08-26 16:06:02,842 INFO sqlalchemy.engine.base.Engine.0x...bbb0 
	CREATE TABLE users (
    	id INTEGER NOT NULL, 
    	name VARCHAR, 
    	fullname VARCHAR, 
    	password VARCHAR, 
    	PRIMARY KEY (id)
	) 

	2009-08-26 16:06:02,858 INFO sqlalchemy.engine.base.Engine.0x...bbb0 ()
	2009-08-26 16:06:02,875 INFO sqlalchemy.engine.base.Engine.0x...bbb0 COMMIT 

使用MetaDATA的create_all方法，并传入engine实例，在数据库中创建表。

	class User(object):
    		def __init__(self,name,fullname,password):
        		self.name=name
        		self.fullname=fullname
        		self.password=password 

    		def __repr__(self):
        		return "<User('%s','%s','%s')>" % (self.name,self.fullname,self.password) 

Table对象定义了数据库的表信息，但没有定义行为，我可以创建一个类来定义行为，这个类是object的子类，类中定义了__init__和__repr__两个方法，这两个方法是可选的，SQLAlchemy从不直接调用__init__()。

	>>> mapper(User,users_table)
	<Mapper at 0x13b4250; User> 

现在通过mapper创建了一个mapper对象将User和users_table进行映射关联。

	>>> ed_user=User('ed','Ed Jones','edspassword')
	>>> ed_user.name
	'ed'
	>>> ed_user.password
	'edspassword'
	>>> str(ed_user.id)
	'None'

上面代码测试了User类，在该类的__init__()方法中没有定义id属性，但id列已经存在于users_table对象，通常情况下mapper会为Table中的所有列创建类属性。 

前面的代码分别建立了Table,Class和Mapper,也可以在一次声明中创建它们. 

	from sqlalchemy import Table,Column,Integer,String,MetaData,ForeignKey
	from sqlalchemy.ext.declarative import declarative_base 

	Base=declarative_base()   #基类 

	class User(Base):
    		__tablename__='users'   #表名  

    		id=Column(Integer,primary_key=True)
    		name=Column(String)
    		fullname=Column(String)
    		password=Column(String) 

    		def __init__(self,name,fullname,password):
        		self.name=name
        		self.fullname=fullname
        		self.password=password 

   		def __repr__(self):
        		return "<User('%s','%s','%s')>" % (self.name,self.fullname,self.password) 

		users_table=User.__table__  #用来获得Table
		metadata=Base.metadata  #获得MetaDATA 

创建Session并绑定一个数据库链接 

	from sqlalchemy.orm import sessionmaker 
	Session=sessionmaker(bind=engine) 

如果没有数据库链接，可以这样创建session 

	Session=sessionmaker() 

当后来由数据库链接后可以这样绑定 

	Session.configure(bind=engine) 

我理解session就是一个事务,它在提交和关闭前，维护着一个数据库链接。 

	>>> ed_user = User(’ed’, ’Ed Jones’, ’edspassword’)
	>>> session.add(ed_user) 

通过session的add方法添加一个对象 

	>>> our_user = session.query(User).filter_by(name=’ed’).first()
		BEGIN
		INSERT INTO users (name, fullname, password) VALUES (?, ?, ?)
		[’ed’, ’Ed Jones’, ’edspassword’]
		SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS user_password
		FROM users
		WHERE users.name = ?
		LIMIT 1 OFFSET 0
		[’ed’]

	>>> our_user
	<User(’ed’,’Ed Jones’, ’edspassword’)>

使用session来查询刚才创建的记录

	>>>ed_user is our_user
	True

实际上Session返回的our_user和ed_user是同一个对象，session对于同一个主关键字只有一个对象与之对应，如果想创建第二个，将会发生错误。

	>>> session.add_all([
	... User(’wendy’, ’Wendy Williams’, ’foobar’),
	... User(’mary’, ’Mary Contrary’, ’xxg527’),
	... User(’fred’, ’Fred Flinstone’, ’blah’)]) 

使用session的add_all方法添加多个记录 

	>>> ed_user.password = ’f8s7ccs’ 

当Ed这个用户发现他的密码不安全时，可以这样修改密码  ：） 

	>>> session.dirty
	IdentitySet([<User(’ed’,’Ed Jones’, ’f8s7ccs’)>]) 

由于修改了ed的密码，所以该记录成为了脏数据，可以这样查看脏数据,session对象很聪明，它什么都知道  ：）

	>>> session.new
	IdentitySet([<User(’wendy’,’Wendy Williams’, ’foobar’)>,
	<User(’mary’,’Mary Contrary’, ’xxg527’)>,
	<User(’fred’,’Fred Flinstone’, ’blah’)>]) 

session还知道刚才添加的3个新数据 

	>>> session.commit()
	UPDATE users SET password=? WHERE users.id = ?
	[’f8s7ccs’, 1]
	INSERT INTO users (name, fullname, password) VALUES (?, ?, ?)
	[’wendy’, ’Wendy Williams’, ’foobar’]
	INSERT INTO users (name, fullname, password) VALUES (?, ?, ?)
	[’mary’, ’Mary Contrary’, ’xxg527’]
	INSERT INTO users (name, fullname, password) VALUES (?, ?, ?)
	[’fred’, ’Fred Flinstone’, ’blah’]
	COMMIT 

现在是时候提交事务并将改变应用到数据库当中了，就这样，很简单吧 ：）

提交后,session将连接释放回连接池，下次需要操作需要新建一个事务了。

	>>> ed_user.id
	BEGIN
	SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.
	FROM users
	WHERE users.id = ?
	[1]1 

现在查看Ed用户的id就不是之前的NONE了，它有了自己的值。 

	>>> ed_user.name = ’Edwardo’ 

	>>> fake_user = User(’fakeuser’, ’Invalid’, ’12345’)
	>>> session.add(fake_user) 

为了继续做实验，我们先修改一个用户的name，再添加一个fake用户 

	>>> session.query(User).filter(User.name.in_([’Edwardo’, ’fakeuser’])).all()
	UPDATE users SET name=? WHERE users.id = ?
	[’Edwardo’, 1]
	INSERT INTO users (name, fullname, password) VALUES (?, ?, ?)
	[’fakeuser’, ’Invalid’, ’12345’]
	SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, user_password
	FROM users
	WHERE users.name IN (?, ?)
	[’Edwardo’, ’fakeuser’][<User(’Edwardo’,’Ed Jones’, ’f8s7ccs’)>, <User(’fakeuser’,’Invalid’,’12345’> 

我们执行一个查询操作会导致前面的的事务被提交。 

	>>> session.rollback()
	ROLLBACK>>> ed_user.name
	BEGIN
	SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users_password
	FROM users
	WHERE users.id = ?
	[1]u’ed’
	>>> fake_user in session
	False 

我们回滚一下，ed_user的名字又变为ed，而fake_user被撤销了。 

	>>> session.query(User).filter(User.name.in_([’ed’, ’fakeuser’])).all()
	SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.p
	FROM users
	WHERE users.name IN (?, ?)
	[’ed’, ’fakeuser’][<User(’ed’,’Ed Jones’, ’f8s7ccs’)>] 

不信的话，再执行以下SELECT查询看看数据库是什么样的。 

	>>> for instance in session.query(User).order_by(User.id):
	... print instance.name, instance.fullname
	SELECT users.id AS users_id, users.name AS users_name,
	users.fullname AS users_fullname, users.password AS users_password
	FROM users ORDER BY users.id
	[]ed Ed Jones
	wendy Wendy Williams
	mary Mary Contrary
	fred Fred Flinstone 

使用session的query方法对数据库执行一下排序查询，并把它们打印出来

	>>> for name, fullname in session.query(User.name, User.fullname):
	... print name, fullname
	SELECT users.name AS users_name, users.fullname AS users_fullname
	FROM users
	[]ed Ed Jones
	wendy Wendy Williams
	mary Mary Contrary
	fred Fred Flinstone 

session的query方法可以接受多种参数，当然也可以这样查询字段 

	>>> for row in session.query(User, User.name).all():
	... print row.User, row.name
	SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users_password
	FROM users
	[]<User(’ed’,’Ed Jones’, ’f8s7ccs’)> ed
	<User(’wendy’,’Wendy Williams’, ’foobar’)> wendy
	<User(’mary’,’Mary Contrary’, ’xxg527’)> mary
	<User(’fred’,’Fred Flinstone’, ’blah’)> fred 

用query方法返回全部记录和name字段。 

	>>> from sqlalchemy.orm import aliased
	>>> user_alias = aliased(User, name=’user_alias’)
	>>> for row in session.query(user_alias, user_alias.name.label(’name_label’)).all():
	... print row.user_alias, row.name_label
	SELECT users_1.id AS users_1_id, users_1.name AS users_1_name, users_1.fullname AS users_1_fullname
	FROM users AS users_1
	[]
	<User(’ed’,’Ed Jones’, ’f8s7ccs’)> ed
	<User(’wendy’,’Wendy Williams’, ’foobar’)> wendy
	<User(’mary’,’Mary Contrary’, ’xxg527’)> mary
	<User(’fred’,’Fred Flinstone’, ’blah’)> fred 

在query方法中使用aliased()和label()，可以创建别名和字段标签 

	>>> for u in session.query(User).order_by(User.id)[1:3]:
	... print u
	SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, user.password AS user_password
	FROM users ORDER BY users.id
	LIMIT 2 OFFSET 1
	[]<User(’wendy’,’Wendy Williams’, ’foobar’)>
	<User(’mary’,’Mary Contrary’, ’xxg527’)> 

这个厉害了，竟然可以对查询的结果做切片操作 

	>>> for name, in session.query(User.name).filter_by(fullname=’Ed Jones’):
	... print name
	SELECT users.name AS users_name FROM users
	WHERE users.fullname = ?
	[’Ed Jones’]ed 

根据字段值条件来查询记录

	>>> for name, in session.query(User.name).filter(User.fullname==’Ed Jones’):
	... print name
	SELECT users.name AS users_name FROM users
	WHERE users.fullname = ?
	[’Ed Jones’]ed 

之前的查询还可以这样写，注意红色部分

	>>> for user in session.query(User).filter(User.name==’ed’).filter(User.fullname==’Ed Jones’）
	... print user
	SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, use
	FROM users
	WHERE users.name = ? AND users.fullname = ?
	[’ed’, ’Ed Jones’]<User(’ed’,’Ed Jones’, ’f8s7ccs’)> 

如果又要查name字段又要查fullname字段，其实就是AND关系啦，两次调用filter就是AND关系啦。 

下面的部分就是常用的过滤操作，很常用的哦 

• equals:

`query.filter(User.name == ’ed’)   #等于`

• not equals:

`query.filter(User.name != ’ed’)  #不等于`

• LIKE:

`query.filter(User.name.like(’%ed%’))  #LIKE`

• IN:

`query.filter(User.name.in_([’ed’, ’wendy’, ’jack’]))  #in操作`

• IS NULL:

`filter(User.name == None)  #IS NULL`

• AND:

	from sqlalchemy import and_
	filter(and_(User.name == ’ed’, User.fullname == ’Ed Jones’))  #AND操作
	filter(User.name == ’ed’).filter(User.fullname == ’Ed Jones’)  #另一种AND操作方式为多次调用filter()或filter_by()

• OR:

	from sqlalchemy import or_
	filter(or_(User.name == ’ed’, User.name == ’wendy’))  #OR操作

• match:

	query.filter(User.name.match(’wendy’))  #匹配，这个目前我不太清楚，不知道和LIKE有啥功能上的区别

	>>> query = session.query(User).filter(User.name.like(’%ed’)).order_by(User.id)
	>>> query.all()
	SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password
	FROM users
	WHERE users.name LIKE ? ORDER BY users.id
	[’%ed’][<User(’ed’,’Ed Jones’, ’f8s7ccs’)>, <User(’fred’,’Fred Flinstone’, ’blah’)>] 

使用query.all()获得一个记录列表。 

	>>>query.first() 

只返回第一条记录 

	>>> from sqlalchemy.orm.exc import MultipleResultsFound
	>>> try:
	... user = query.one()
	... except MultipleResultsFound, e:
	... print e 

	[’%ed’]Multiple rows were found for one()

这个one()我就不太理解了，你要知道请留言告诉我，我从精神上感谢你，但是one()是会引发异常的，小心使用！

	>>> from sqlalchemy.orm.exc import NoResultFound
	>>> try:
	... user = query.filter(User.id == 99).one()
	... except NoResultFound, e:
	... print e 

	[’%ed’, 99]No row was found for one()

这个和上面的一样，我不知道，嘿嘿，你知道一定要告诉我哦~

	>>> for user in session.query(User).filter("id<224").order_by("id").all():
	... print user.name 

在filter()和orader_by()中可以直接使用SQL字符串。 

	>>> session.query(User).filter("id<:value and name=:name").\
	... params(value=224, name=’fred’).order_by(User.id).one() 

可以使用params()来传递参数

	>>> session.query(User).from_statement("SELECT * FROM users where name=:name").params(name=’ed’).all()

这个就对于熟悉SQL的用户很方便，不过你要保证SQL语句中的字段名是被映射过的。

现在我们该引入第2个表了，每个用户都可能有多个email地址与用户名关联，这是一个典型的1对多的关系，下面先创建这个包含email地址的addresses表

	>>> from sqlalchemy import ForeignKey
	>>> from sqlalchemy.orm import relation, backref
	>>> class Address(Base):
	... __tablename__ = ’addresses’
	... id = Column(Integer, primary_key=True)
	... email_address = Column(String, nullable=False)
	... user_id = Column(Integer, ForeignKey(’users.id’))
	...
	... user = relation(User, backref=backref(’addresses’, order_by=id))
	...
	... def __init__(self, email_address): 

	... self.email_address = email_address
	...
	... def __repr__(self):
	... return "<Address(’%s’)>" % self.email_address 

代码的缩进格式已经完蛋了，索性不管他了，我想缩进不会难倒任何python程序员的，这段代码大部分已经在前面学习了，黑体部分是现在要学习的。

其中上面的代码是在数据库级别在两个表之间建立关系，ForeignKey增加一个user_id来关联users.id属性,建立从Address到User的多对1关系，增加一个user属性，同样给User类增加一个addresses集合，backref建立从user从User到Address是1对多的关系。

	class User(Base):
	# ....
	addresses = relation(Address, order_by=Address.id, backref="user") 

	relation为User类定义一个关系。

	>>> metadata.create_all(engine)
	PRAGMA table_info("users")
	()
	PRAGMA table_info("addresses")
	()
	CREATE TABLE addresses (
	id INTEGER NOT NULL,
	email_address VARCHAR NOT NULL,
	user_id INTEGER,
	PRIMARY KEY (id),
	FOREIGN KEY(user_id) REFERENCES users (id)
	)
	()
	COMMIT 

再次create_all()，已经存在的表将被忽略。

	>>> jack = User(’jack’, ’Jack Bean’, ’gjffdd’)
	>>> jack.addresses
	[] 

现在只有一个有数据的User和没有数据的Addresses，查询后就是这个样子的 :( 

	>>> jack.addresses = [Address(email_address=’jack@google.com’), Address(email_address=’j25@yahoo.com’) 

可以像这样为用户添加一个或多个邮箱。 

	>>> jack.addresses[1]
	<Address(’j25@yahoo.com’)>

这样就可以返回一个与用户关联的邮箱地址

	>>> jack.addresses[1].user
	<User(’jack’,’Jack Bean’, ’gjffdd’)>

通过邮箱地址还可以返回这个用户呢！赞！

	>>> session.add(jack)
	>>> session.commit()
	INSERT INTO users (name, fullname, password) VALUES (?, ?, ?)
	[’jack’, ’Jack Bean’, ’gjffdd’]
	INSERT INTO addresses (email_address, user_id) VALUES (?, ?)
	[’jack@google.com’, 5]
	INSERT INTO addresses (email_address, user_id) VALUES (?, ?)
	[’j25@yahoo.com’, 5]
	COMMIT 

先把提交jack到数据库中，这将会导致插入1个用户记录和两个相关的邮箱记录。

	>>> jack = session.query(User).filter_by(name=’jack’).one()

现在查询一下数据库里的jack记录

	>>> jack.addresses

再通过查询出的jack来查询相关联的邮箱记录

	>>> from sqlalchemy.orm import eagerload
	>>> jack = session.query(User).options(eagerload(’addresses’)).filter_by(name=’jack’).one()

	>>> jack
	<User(’jack’,’Jack Bean’, ’gjffdd’)>
	>>> jack.addresses
	[<Address(’jack@google.com’)>, <Address(’j25@yahoo.com’)>]

这段代码其实和之前的两段代码干的一个事，不过之前的代码的user和address是在取值时才从数据库取数据，而这段代码是一次取回数据的。

	>>> for u, a in session.query(User, Address).filter(User.id==Address.user_id).\
	... filter(Address.email_address==’jack@google.com’).all():
	... print u, a 

这个好理解，就是执行一个多表查询。

	>>> from sqlalchemy.orm import join
	>>> session.query(User).select_from(join(User, Address)).\
	... filter(Address.email_address==’jack@google.com’).all()
	SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, user.password=user_password
	FROM users JOIN addresses ON users.id = addresses.user_id
	WHERE addresses.email_address = ?
	[’jack@google.com’][<User(’jack’,’Jack Bean’, ’gjffdd’)>] 

**使用join()来执行JOIN查询。**

join()知道如何join表User和Address，因为这两表之间只有一个外键，如果他们咩有外键或有多个外键，则需要这样设置join()

	join(User, Address, User.id==Address.user_id) # 描述关系

	join(User, Address, User.addresses) # 从左到右指定relation

	join(User, Address, ’addresses’) # 同上，这是字符串表示方法

	>>> session.query(User).join(User.addresses).\
	... filter(Address.email_address==’jack@google.com’).all() 

	[’jack@google.com’][<User(’jack’,’Jack Bean’, ’gjffdd’)>]

这是join()一种更简单的写法。

	session.query(User).join((Address, User.addresses))

join()的其他写法

	session.query(Foo).join(Foo.bars, Bar.bats, (Bat, ’widgets’))

这是多个JOIN的表示方法，SQL类似这样：foo JOIN bars ON <onclause> JOIN bats ON<onclause> JOIN widgets ON <onclause>

	>>> from sqlalchemy.orm import aliased
	>>> adalias1 = aliased(Address)
	>>> adalias2 = aliased(Address)
	>>> for username, email1, email2 in \
	... session.query(User.name, adalias1.email_address, adalias2.email_address).\
	... join((adalias1, User.addresses), (adalias2, User.addresses)).\ 

	... filter(adalias1.email_address==’jack@google.com’).\
	... filter(adalias2.email_address==’j25@yahoo.com’):
	... print username, email1, email2
	SELECT users.name AS users_name, addresses_1.email_address AS addresses_1_email_address,
	addresses_2.email_address AS addresses_2_email_address
	FROM users JOIN addresses AS addresses_1 ON users.id = addresses_1.user_id
	JOIN addresses AS addresses_2 ON users.id = addresses_2.user_id
	WHERE addresses_1.email_address = ? AND addresses_2.email_address = ?
	[’jack@google.com’, ’j25@yahoo.com’]jack jack@google.com j25@yahoo.com 

使用别名同时来获取多个不同的值

	SELECT users.*, adr_count.address_count FROM users LEFT OUTER JOIN
	(SELECT user_id, count(*) AS address_count FROM addresses GROUP BY user_id) AS adr_count
	ON users.id=adr_count.user_id 

上面的子查询SQL可以写作下面的代码 

	>>> from sqlalchemy.sql import func
	>>> stmt = session.query(Address.user_id, func.count(’*’).label(’address_count’)).group_by(Address.user_id)  

	>>> for u, count in session.query(User, stmt.c.address_count).\
	... outerjoin((stmt, User.id==stmt.c.user_id)).order_by(User.id):
	... print u, count
	SELECT users.id AS users_id, users.name AS users_name,
	users.fullname AS users_fullname, users.password AS users_password,
	anon_1.address_count AS anon_1_address_count
	FROM users LEFT OUTER JOIN (SELECT addresses.user_id AS user_id, count(?) AS address_count
	FROM addresses GROUP BY addresses.user_id) AS anon_1 ON users.id = anon_1.user_id
	ORDER BY users.id
	[’*’]<User(’ed’,’Ed Jones’, ’f8s7ccs’)> None
	<User(’wendy’,’Wendy Williams’, ’foobar’)> None
	<User(’mary’,’Mary Contrary’, ’xxg527’)> None
	<User(’fred’,’Fred Flinstone’, ’blah’)> None
	<User(’jack’,’Jack Bean’, ’gjffdd’)> 2

左外连接

	>>> stmt = session.query(Address).filter(Address.email_address != ’j25@yahoo.com’).subquery()
	>>> adalias = aliased(Address, stmt)
	>>> for user, address in session.query(User, adalias).join((adalias, User.addresses)):
	... print user, address
	SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname,
	users.password AS users_password, anon_1.id AS anon_1_id,
	anon_1.email_address AS anon_1_email_address, anon_1.user_id AS anon_1_user_id
	FROM users JOIN (SELECT addresses.id AS id, addresses.email_address AS email_address, addresses.user_id 
	FROM addresses
	WHERE addresses.email_address != ?) AS anon_1 ON users.id = anon_1.user_id
	[’j25@yahoo.com’]<User(’jack’,’Jack Bean’, ’gjffdd’)> <Address(’jack@google.com’)> 

在子查询中使用别名

	>>> from sqlalchemy.sql import exists
	>>> stmt = exists().where(Address.user_id==User.id)
	>>> for name, in session.query(User.name).filter(stmt):
	... print name
	SELECT users.name AS users_name
	FROM users
	WHERE EXISTS (SELECT *
	FROM addresses
	WHERE addresses.user_id = users.id)
	[]jack 

EXISTS关键字的使用

	>>> for name, in session.query(User.name).filter(User.addresses.any()):
	... print name
	SELECT users.name AS users_name
	FROM users
	WHERE EXISTS (SELECT 1
	FROM addresses
	WHERE users.id = addresses.user_id)
	[]jack 

any()的使用 

	>>> for name, in session.query(User.name).\
	... filter(User.addresses.any(Address.email_address.like(’%google%’))): 

	... print name
	SELECT users.name AS users_name
	FROM users
	WHERE EXISTS (SELECT 1
	FROM addresses
	WHERE users.id = addresses.user_id AND addresses.email_address LIKE ?)
	[’%google%’]jack 

any()的使用

	>>> session.query(Address).filter(~Address.user.has(User.name==’jack’)).all()
	SELECT addresses.id AS addresses_id, addresses.email_address AS addresses_email_address,
	addresses.user_id AS addresses_user_id
	FROM addresses
	WHERE NOT (EXISTS (SELECT 1
	FROM users
	WHERE users.id = addresses.user_id AND users.name = ?))
	[’jack’][] 

has()用法

常用的关系操作，如下部分：

• equals (用于多对一):

`query.filter(Address.user == someuser)`

• not equals (用于多对一):

`query.filter(Address.user != someuser)`

• IS NULL (用于多对一):

`query.filter(Address.user == None)`

• contains (用于一对多和多对多集合):

`query.filter(User.addresses.contains(someaddress))`

• any (用于一对多和多对多集合):

`query.filter(User.addresses.any(Address.email_address == ’bar’))`

# 提供关键字参数:

`query.filter(User.addresses.any(email_address=’bar’))`

• has (用于多对一 ):

`query.filter(Address.user.has(name=’ed’))`

• with_parent (用于任何关系):

`session.query(Address).with_parent(someuser, ’addresses’) `

	>>> session.delete(jack)
	>>> session.query(User).filter_by(name=’jack’).count()
	UPDATE addresses SET user_id=? WHERE addresses.id = ?
	[None, 1]
	UPDATE addresses SET user_id=? WHERE addresses.id = ?
	[None, 2]
	DELETE FROM users WHERE users.id = ?
	[5]
	SELECT count(1) AS count_1
	FROM users
	WHERE users.name = ?
	[’jack’]0 

删除jack，并统计删除后的user表记录数 

	>>> session.query(Address).filter(
	... Address.email_address.in_([’jack@google.com’, ’j25@yahoo.com’])
	... ).count()
	SELECT count(1) AS count_1
	FROM addresses
	WHERE addresses.email_address IN (?, ?)
	[’jack@google.com’, ’j25@yahoo.com’]2 

删除jack后再查询一下Address表，看看jack关联的邮箱记录是否存在，哦，他们依然存在，可恶！他们的user_id仅仅是被赋值为NULL

	>>> session.close() # 回滚并关闭事务
	>>> from sqlalchemy.orm import clear_mappers
	>>> clear_mappers() #清除映射

为了能在删除用户的时候删除关联的邮箱记录，需要关闭session然后清除映射，并重新配置

	>>> mapper(User, users_table, properties={
	... ’addresses’:relation(Address, backref=’user’, cascade="all, delete, delete-orphan") 
	... })
	<Mapper at 0x...; User>
	>>> addresses_table = Address.__table__
	>>> mapper(Address, addresses_table)
	<Mapper at 0x...; Address> 

下面重新配置映射，关键是cascade的值，然后重新映射Address和addresses_table 

**load Jack by primary key**

	>>> jack = session.query(User).get(5)
	BEGIN
	SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password
	FROM users
	WHERE users.id = ?
	[5]# remove one Address (lazy load fires off)
	>>> del jack.addresses[1]
	SELECT addresses.id AS addresses_id, addresses.email_address AS addresses_email_address, addresses.user
	FROM addresses
	WHERE ? = addresses.user_id
	[5]# only one address remains
	>>> session.query(Address).filter(
	... Address.email_address.in_([’jack@google.com’, ’j25@yahoo.com’])
	... ).count()
	DELETE FROM addresses WHERE addresses.id = ?
	[2]
	SELECT count(1) AS count_1
	FROM addresses
	WHERE addresses.email_address IN (?, ?)
	[’jack@google.com’, ’j25@yahoo.com’]1 

通过重新配置并映射，现在删除jack的addresses集合里的信箱地址时，数据库里也会被删除掉。

	>>> session.delete(jack)
	>>> session.query(User).filter_by(name=’jack’).count()
	DELETE FROM addresses WHERE addresses.id = ?
	[1]
	DELETE FROM users WHERE users.id = ?
	[5]
	SELECT count(1) AS count_1
	FROM users
	WHERE users.name = ?
	[’jack’]0 

删除jack这个用户，剩余的相关邮箱记录也被删除

	>>> session.query(Address).filter(
	... Address.email_address.in_([’jack@google.com’, ’j25@yahoo.com’])
	... ).count()
	SELECT count(1) AS count_1
	FROM addresses
	WHERE addresses.email_address IN (?, ?)
	[’jack@google.com’, ’j25@yahoo.com’]0 

再查询一下数据库，看看jack的信箱记录是否还存在？ 
如果要写一个BLOG程序，就会用到多对多的关系，比如：日志和关键字之间的关系 

	>>> from sqlalchemy import Text
	>>> # 关系表
	>>> post_keywords = Table(’post_keywords’, metadata,
	... Column(’post_id’, Integer, ForeignKey(’posts.id’)),
	... Column(’keyword_id’, Integer, ForeignKey(’keywords.id’))
	... )

	>>> class BlogPost(Base):   #日志类
	... __tablename__ = ’posts’
	...
	... id = Column(Integer, primary_key=True)
	... user_id = Column(Integer, ForeignKey(’users.id’))
	... headline = Column(String(255), nullable=False)
	... body = Column(Text)
	...
	... # many to many BlogPost<->Keyword
	... keywords = relation(’Keyword’, secondary=post_keywords, backref=’posts’)
	...
	... def __init__(self, headline, body, author):
	... self.author = author
	... self.headline = headline
	... self.body = body
	...
	... def __repr__(self):
	... return "BlogPost(%r, %r, %r)" % (self.headline, self.body, self.author) 

	>>> class Keyword(Base):   #关键字类
	... __tablename__ = ’keywords’
	...
	... id = Column(Integer, primary_key=True)
	... keyword = Column(String(50), nullable=False, unique=True)
	...
	... def __init__(self, keyword):
	... self.keyword = keyword 

	>>> from sqlalchemy.orm import backref
	>>> # "dynamic" loading relation to User
	>>> BlogPost.author = relation(User, backref=backref(’posts’, lazy=’dynamic’))  #动态设置关系 

	>>> metadata.create_all(engine)  #创建表
	PRAGMA table_info("users")
	()
	PRAGMA table_info("addresses")
	()
	PRAGMA table_info("posts")
	()
	PRAGMA table_info("keywords")
	()
	PRAGMA table_info("post_keywords")
	()
	CREATE TABLE posts (
	id INTEGER NOT NULL,
	user_id INTEGER,
	headline VARCHAR(255) NOT NULL,
	body TEXT,
	PRIMARY KEY (id),
	FOREIGN KEY(user_id) REFERENCES users (id)
	)
	()
	COMMIT
	CREATE TABLE keywords (
	id INTEGER NOT NULL,
	keyword VARCHAR(50) NOT NULL,
	PRIMARY KEY (id),
	UNIQUE (keyword)
	)
	() 

	COMMIT
	CREATE TABLE post_keywords (
	post_id INTEGER,
	keyword_id INTEGER,
	FOREIGN KEY(post_id) REFERENCES posts (id),
	FOREIGN KEY(keyword_id) REFERENCES keywords (id)
	)
	()
	COMMIT

**下面先写一个日志**

	>>> wendy = session.query(User).filter_by(name=’wendy’).one()
	SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password FROM users
	WHERE users.name = ?
	LIMIT 2 OFFSET 0
	[’wendy’]>>> post = BlogPost("Wendy’s Blog Post", "This is a test", wendy)
	>>> session.add(post) 

**给日志添加两个关键字** 

	>>> post.keywords.append(Keyword(’wendy’))
	>>> post.keywords.append(Keyword(’firstpost’)) 

**查找一个包含firstpost的日志**

	>>> session.query(BlogPost).filter(BlogPost.keywords.any(keyword=’firstpost’)).all()
	INSERT INTO keywords (keyword) VALUES (?)
	[’wendy’]
	INSERT INTO keywords (keyword) VALUES (?)
	[’firstpost’]
	INSERT INTO posts (user_id, headline, body) VALUES (?, ?, ?)
	[2, "Wendy’s Blog Post", ’This is a test’]
	INSERT INTO post_keywords (post_id, keyword_id) VALUES (?, ?)
	[[1, 2], [1, 1]]
	SELECT posts.id AS posts_id, posts.user_id AS posts_user_id, posts.headline AS posts_headline, posts.bod
	FROM posts
	WHERE EXISTS (SELECT 1
	FROM post_keywords, keywords
	WHERE posts.id = post_keywords.post_id AND keywords.id = post_keywords.keyword_id AND keywords.keyword =
	[’firstpost’][BlogPost("Wendy’s Blog Post", ’This is a test’, <User(’wendy’,’Wendy Williams’, ’foobar’)> 

**查找作者为wendy的日志**

	>>> session.query(BlogPost).filter(BlogPost.author==wendy).\
	... filter(BlogPost.keywords.any(keyword=’firstpost’)).all()
	SELECT posts.id AS posts_id, posts.user_id AS posts_user_id, posts.headline AS posts_headline, posts.bod
	FROM posts
	WHERE ? = posts.user_id AND (EXISTS (SELECT 1
	FROM post_keywords, keywords
	WHERE posts.id = post_keywords.post_id AND keywords.id = post_keywords.keyword_id AND keywords.keyword =
	[2, ’firstpost’][BlogPost("Wendy’s Blog Post", ’This is a test’, <User(’wendy’,’Wendy Williams’, ’foobar 

**用wendy来查找** 

	>>> wendy.posts.filter(BlogPost.keywords.any(keyword=’firstpost’)).all()
	SELECT posts.id AS posts_id, posts.user_id AS posts_user_id, posts.headline AS posts_headline, posts.bod
	FROM posts
	WHERE ? = posts.user_id AND (EXISTS (SELECT 1
	FROM post_keywords, keywords
	WHERE posts.id = post_keywords.post_id AND keywords.id = post_keywords.keyword_id AND keywords.keyword =
	[2, ’firstpost’][BlogPost("Wendy’s Blog Post", ’This is a test’, <User(’wendy’,’Wendy Williams’, ’foobar 

这篇日志好长呀，写了3个小时，一些代码自己也动手测试了，今天的收获就是对SQLAlchemy的概貌有了一个了解，可以创建表(类)、映射、session,以及使用session做一些query。

今天只学习了38页，还有250页的资料再等着我，希望我能坚持下去！！！

参考资料：

《SQLAlchemy Documentation》
Release 0.5.4 

转载请注明来源，谢谢~ 

[http://booolee.cnblogs.com/ ](http://booolee.cnblogs.com/ )

