# Chapter8 标准IO库

## 8.1 面向对象的标准库

| 头文件   | 类型                                                         |
| :------- | ------------------------------------------------------------ |
| iostream | istream从流中读取<br />ostream写到流中去<br />iostream对流进行读写；从istream和ostream派生而来 |
| fstream  | ifstream从文件中读取；由istream派生而来<br />ofstream写到文件中去；由ostream派生而来<br />fstream对流进行读写；由iostream派生而来 |
| sstream  | istringstream从string对象中读取；由istream派生而来<br />ostringstream写到string对象中去；由ostream派生而来<br />stringstream对string对象进行读写；由iostream派生而来 |

对IO类型使用继承还有另外一层含义：如果函数有基类类型的引用形参时，可以给函数传递其派生类的对象。

IO对象不可复制或赋值：

1. 由于流不能复制，因此不能存储在vector（或其他）容器中（即不存在能存储流对象的vector或其他容器）。
2. 形参或返回类型也不能为流类型。如果需要返回或传递IO对象，必须传递或返回指向对象的指针或引用（非const引用）。

## 8.2 条件状态

标准IO库的条件状态

| strm::iostate    | 机器相关的整形名，由各个iostream类定义，用于定义条件状态     |
| ---------------- | ------------------------------------------------------------ |
| strm::badbit     | strm::iostate类型的值，用于指出被破坏的流                    |
| strm::failbit    | strm::iostate类型的值，用于指出失败的IO操作                  |
| strm::eofbite    | strm::iostate类型的值，用于指出流已经到达文件的结束符        |
| s.eof()          | 如果设置了流s的eofbit值，则该函数返回true                    |
| s.fail()         | 如果设置了流s的failbit值，则该函数返回true                   |
| s.bad()          | 如果设置了流s的badbit值，则该函数返回true                    |
| s.good()         | 如果流s出于有效状态，该函数返回true                          |
| s.clear()        | 将流s中的所有状态都重设为有效状态                            |
| s.s.clear(flag)  | 将流s中的某个指定条件状态设置为有效。flag的类型是strm::iostate |
| s.setstate(flag) | 给流添加指定条件。flag的类型是strm::iostate                  |
| s.rdstate()      | 返回流s的当前条件，返回值类型为strm::iostate                 |

1. 条件状态

   badbit:标志着系统级别的错误，如无法恢复的读写错误。

   failbit:可恢复的错误，如希望获得整形数值时输入了字符。

   eofbit:文件结束符，遇到文件结束符是设置的，此时还设置了failbit。

逗号操作符：首先计算他的每一个操作数，然后返回最右边的操作数作为整个操作的结果。

2. 流状态的查询和控制

   ~~~
   int ival;
   while(cin>>ival, !cin.eof()){
   	if(cin.bad()){
   		throw runtime_error("IO stream corrupted");
   	}
   	if(cin.fail()){
   	cerr<<"  ";
   	cin.clear(istream::failbit);
   	cin.ignore(std::numeric_limits<std::streamsize>::max(),'\n');
   	continue;
   	}
   }
   ~~~

3. 条件状态的访问

   rdstate成员函数返回一个iostate类型的值，该值对应于流当前的整个条件状态；

   ~~~ 
   istream::iostate old_state = cin.rdstate();
   cin.clear();
   process_input();//使用cin
   cin.clear(old_state);//重设cin为old_state
   ~~~

4. 多种状态的处理

   ~~~
   is.setstate(ifstream::badbit|ifstream::failbit);//将对象is的failbit和badbit同时打开
   ~~~

## 8.3 输出缓冲区的管理

导致缓冲区的内容将被刷新：

1. 程序正常结束。作为main返回工作的一部分，将清空所有的缓冲区
2. 在一些不确定的时候，缓冲区可能已经满了，在这种情况下，缓冲区将会在写下一个值之前刷新。4
3. 用操作符显示的刷新缓冲区，例如行结束符endl;
4. 在每次输出操作执行完后，用unitbuff设置流的内部状态，从而清空缓冲区
5. 可将输出流与输入流关联(tie)起来，此时，在读入流时 将刷新其关联的输出缓冲区

**输出缓冲区的刷新**

使用endl或者flush操纵符：

~~~
cout<<"hi"<<flush;//刷新缓冲区
cout<<"hi"<<ends;//插入一个null,刷新缓冲区
cout<<"hi"<<endl;//插入新行，刷新缓冲区
~~~

**unitbuff操作符**

刷新所有输出，最好使用unitbuff操纵符。这个操纵符在每次执行完写后都刷新流：

~~~ 
cout<<unibuff<<"first"<<"second"<<nounitbuff;
//等价于
cout<<"first"<<flush<<"second"<<flush;
~~~

unitbuff操纵符将流恢复为使用正常的、由系统管理的缓冲区刷新方式。

**将输入输出关联在一起**

~~~
cin.tie(&cout);//将cin和cout关联
ostream *old_tie = cin.tie();
cin,tie(0); //打破关联，cin和cout不在关联在一起
cin.tie(&cerr);//cin和cerr关联在一起，不必要

cin.tie(0);//解除cin和Cerr关联
cin.tie(old_tie);//重建cin和cout关联
~~~



## 8.4 文件的输入和输出

### 	8.4.1 文件流的使用

~~~
ifstream infile;
ofstream outfile;

infile.open("in");
outfile.open("out");
//c++文件名要使用C风格字符串（c_str()函数）

if(infile){ };
if(outfile){ };//检查文件是否打开成功
~~~

1. 将文件流与新文件重新绑定(先调用close，之后在带打开新文件)

   ~~~
   ifstream infile("in");
   infile.close();
   infile.open("next");
   ~~~

2. 清除文件流的状态

   如果打算重用已存在的流对象，那么while循环必须在每次循环关闭的时记得关闭（close）和清空（clear）文件流：

   ~~~
   ifstream input;
   vector<string>::count_iterator it = files.begin();
   while(it!=files.end()){
   	input.open(it->c_str());
   	if(!input)
   		break;
   	while(input>>s)
   		process(s);
   	input.close();
   	input.clear();
   	++it;
   }//如果忽略clear调用，则只能读入第一个文件。因为一旦input处于错误状态，在没有clear之前，任何在input上的操作都会失败。
   ~~~

### 8.4.2 文件模式

在打开文件时，无论是调用open还是以文件名作为流初始化的一部分，都需指定文件模式。

| 文件模式 |                                  |
| -------- | -------------------------------- |
| in       | 打开文件做读操作                 |
| out      | 打开文件做写操作                 |
| app      | 在每次写文件之前找到文件尾       |
| ate      | 打开文件后立即将文件定位在文件尾 |
| trunc    | 打开文件时清空已存在的文件流     |
| binary   | 以二进制形式进行IO操作           |

默认时，与ifstream流关联的文件将以in模式打开；与ofstream关联的文件将以out模式打开，将会清除该文件之前存储的所有数据。

~~~
ofstream appfile("file2", ofstream::app);//保存打开文件中已存在的数据
~~~

1. 对同一文件作输入和输出运算

   默认情况下，fstream对象以in和out模式同时打开文件，当文件以in和out同时打开时不会清空。

   ~~~
   fstream input("copyOut", fstream::in|fstream::out);
   ~~~

2. 模式是文件的属性，不是流的属性。

   ==每次打开文件时都会设置模式==

3. 打开模式的有效组合

   | 文件模式的组合 |                                            |
   | -------------- | ------------------------------------------ |
   | out            | 打开文件做写操作，删除文件中已有的数据     |
   | out\|app       | 打开文件做写操作，在文件尾写入             |
   | out\|trunc     | 与out模式相同                              |
   | in             | 打开文件做读操作                           |
   | in\|out        | 打开文件做读、写操作，并定位于文件开头处   |
   | in\|out\|trunc | 打开文件做读、写操作，删除文件中已有的数据 |

### 8.4.3 一个打开并检查输入文件的程序

~~~~
ifstream& open_file(ifstream &in, const string &file){
	in.close();
	in.clear();
	
	in.open(file.c_str());
	return in;//in要么正确绑定文件，要么为错误状态
}
~~~~

## 8.5 字符串流

| stringstream特定的操作 |                                                            |
| ---------------------- | ---------------------------------------------------------- |
| stringstream strm；    | 创建自由的stringstream对象                                 |
| stringstream strm(s)； | 创建存储s的副本的stringstream对象，其中s是string类型的对象 |
| strm.str()             | 返回strm中存储的string类型对象                             |
| strm.str(s)            | 将string类型中的s复制给strm，返回void                      |

1. stringstream对象的使用

   ~~~
   string line,word;
   while(getline(cin,line)){
   
   	istringstream stream(line);//处理每一行
       
       while(stream>>word){
       //处理每一个单词
       }
   }
   ~~~

2. stringstream提供的转换和/或格式化

   string流可以像标准输入输出流一样，自动判别数据类型输出，遇到空格停止；可以方便用来实现格式转换。

   定义： stringstream ss；  //定义了一个string流，可以输入也可以输出；

   ~~~
   ss<<"carea 89 M 65.3";    //初始化在流里面的数据，
    string name;
    int age;
    char sex;
    float weight;
    ss>>name>>age>>sex>>weight; //输出的时候将流中数据输出到变量中；
    cout<<"姓名："<<name<<endl
    <<"年龄："<<age<<endl
    <<"性别："<<sex<<endl
    <<"体重："<<weight<<endl;
   ~~~

   也可以单独定义输入流（其实是输出流，是将一个string串中的数据按不同的数据类型输出到变量中；）

   ~~~
   string s="Hello world!";
   
   string a；
   
   istringstream sin(s);    //用s来初始化输入流，流中现在存在的是s中的内容；
   
   sin>>a；           //将“Hello”输出到a中；
   ~~~

   **一般情况下，适用输入操作符读string对象时，空白符会被忽略。**

   

    