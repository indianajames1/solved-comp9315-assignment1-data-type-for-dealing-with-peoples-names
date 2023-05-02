Download Link: https://assignmentchef.com/product/solved-comp9315-assignment1-data-type-for-dealing-with-peoples-names
<br>
This assignment aims to give you an understanding of how data is treated inside a DBMS practice in adding a new base type to PostgreSQL.The goal is to implement a new data type for PostgreSQL, complete with input/output functions, comparison operators, formatting functions, and the ability to build indexes on values of the type.

We use the following names in the discussion below

<em>PG_CODE</em> … the directory where your PostgreSQL source code is located   (typically

/srvr/<em>YOU</em>/postgresql-12.1/)

<em>PG_HOME</em> … the directory where you have installed the PostgreSQL binaries   (typically

/srvr/<em>YOU</em>/pgsql/bin/)

<em>PG_DATA</em> … the directory where you have placed PostgreSQL’s data   (typically

/srvr/<em>YOU</em>/pgsql/data/)

<em>PG_LOG</em> … the file where you send PostgreSQL’s log output   (typically /srvr/<em>YOU</em>/pgsql/data/log)

<h1>Introduction</h1>

PostgreSQL has an extensibility model which, among other things, provides a well-defined process for adding new data types into a PostgreSQL server. This capability has led to the development by PostgreSQL users of a number of types (such as polygons) which have become part of the standard distribution. It also means that PostgreSQL is the database of choice in research projects which aim to push the boundaries of what kind of data a DBMS can manage.

In this assignment, we will be adding a new data type for dealing with people’s names. “Hmmm”, you say, “but aren’t they just text strings, typically implemented as two attributes, one for family name and one for given names?”. That may be true, but making names into a separate base data type allows us to explore how we store and manipulate them.

One common way of writing names (e.g. used in UNSW student systems) is

Shepherd,John Andrew

Lin,Xuemin

Eilish,Billie

Martin, Eric Andre

Lakshminarasimhan,Venkateswaran Chandrasekara

Marshall-Martin, Sally Angela

Featherstone,Albert Basil Ernest George Harold Randolph William i.e.

<em>FamilyName</em>,<em>GivenNames</em>

We give a more precise description of what text strings are valid PersonNames below.

<strong>Adding Data Types in PostgreSQL</strong>

The process for adding new base data types in PostgreSQL is described in the following sections of the PostgreSQL documentation:

<a href="https://cgi.cse.unsw.edu.au/~cs9315/20T1/postgresql/documentation/xtypes.html">37.13 User-defined Types</a>

<a href="https://cgi.cse.unsw.edu.au/~cs9315/20T1/postgresql/documentation/xfunc-c.html">37.10 C-Language Functions</a>

<a href="https://cgi.cse.unsw.edu.au/~cs9315/20T1/postgresql/documentation/xoper.html">37.14 User-defined Operators</a>

<a href="https://cgi.cse.unsw.edu.au/~cs9315/20T1/postgresql/documentation/sql-createtype.html">SQL: CREATE TYPE</a>

<a href="https://cgi.cse.unsw.edu.au/~cs9315/20T1/postgresql/documentation/sql-createoperator.html">SQL: CREATE OPERATOR</a>

<a href="https://cgi.cse.unsw.edu.au/~cs9315/20T1/postgresql/documentation/sql-createopclass.html">SQL: CREATE OPERATOR CLASS</a>

Section 37.13 uses an example of a complex number type, which you can use as a starting point for defining your PersonName data type (see below). There are other examples of new data types under the directories:

<em>PG_CODE</em>/contrib/chkpass/ … an auto-encrypted password datatype

<em>PG_CODE</em>/contrib/citext/ … a case-insensitive character string datatype

<em>PG_CODE</em>/contrib/seg/ … a confidence-interval datatype

These may or may not give you some useful ideas on how to implement the PersonName data type. For example, many of these data types are fixed-size, while PersonNames are variable-sized. A potentially useful example of implementing variable-sized types can be found in:

<em> PG_CODE</em>/src/tutorial/funcs.c … implementation of several data types

<h1>Setting Up</h1>

You ought to start this assignment with a fresh copy of PostgreSQL, without any changes that you might have made for the Prac exercises (unless these changes are trivial). Note that you only need to configure, compile and install your PostgreSQL server once for this assignment. All subsequent compilation takes place in the src/tutorial directory, and only requires modification of the files there.

Once you have re-installed your PostgreSQL server, you should run the following commands:

$ <strong>cd <em>PG_CODE</em>/src/tutorial</strong>

$ <strong>cp complex.c pname.c</strong>

$ <strong>cp complex.source pname.source</strong>

Note the pname.* files will contain <em>many</em> references to complex; I do not want to see any remaining occurences of the word complex in the files that you eventually submit. These files simply provide a template in which you create <em>your</em> PersonName type.

Once you’ve made the pname.* files, you should also edit the Makefile in this directory and add the green text to the following lines:

MODULES = complex funcs pname

DATA_built = advanced.sql basics.sql complex.sql funcs.sql syscat.sql pname.sql

The rest of the work for this assignment involves editing only the pname.c and pname.source files. In order for the Makefile to work properly, you must use the identifier _OBJWD_ in the pname.source file to refer to the directory holding the compiled library. You should never modify directly the pname.sql file produced by the Makefile. Place <em>all</em> of your C code in the pname.c file; do not create any other *.c files.

Note that your submitted versions of pname.c and pname.source should not contain any references to the complex type. Make sure that the documentation (comments in program) describes the code that <em>you </em>wrote. Leaving the word complex anywhere in a pname.* file will cost 1 mark.

<h1>The Person Name Data Type</h1>

We wish to define a new base type PersonName to represent people’s names, in the format

<em>FamilyName</em>,<em>GivenNames</em>. We also aim to define a useful set of operations on values of type

PersonName and wish to be able to create indexes on PersonName attributes. How you represent PersonName values internally, and how you implement the functions to manipulate them internally, is up to you. However, they must satisfy the requirements below.

Once implemented correctly, you should be able to use your PostgreSQL server to build the following kind of SQL applications:

<table width="642">

 <tbody>

  <tr>

   <td width="642">create table Students (    zid       integer primary key,    name      PersonName not null,    degree    text,— etc. etc.);insert into Students(zid,name,degree) values(9300035,’Shepherd, John Andrew’, ‘BSc(Computer Science)’), (5012345,’Smith, Stephen’, ‘BE(Hons)(Software Engineering)’); create index on Students using hash (name); select a.zid, a.name, b.zidfrom   Students a join Students b on (a.name = b.name); select family(name), given(name), show(name) from   Students; select name,count(*) from   Students group  by name;</td>

  </tr>

 </tbody>

</table>

Having defined a hash-based file structure, we would expect that the queries would make use of it. You can check this by adding the keyword EXPLAIN before the query, e.g.

db=# <strong>explain analyze select * from Students where name=’Smith,John’;</strong>

which should, once you have correctly implemented the data type and loaded sufficient data, show that an index-based scan of the data is being used. Note that this will only be evident if you use a large amount of data (e.g. one of the larger test data samples to be provided).

<h2>Person Name values</h2>

Valid PersonNames will have the above format with the following qualifications:

there may be a single space after the comma

there will be <strong>no</strong> people with just one name (e.g. <em>no</em> Prince, Jesus, Aristotle, etc.) there will be <strong>no</strong> numbers (e.g. <em>no</em>Gates, William 3rd) there will be <strong>no</strong> titles (e.g. <em>no</em> Dr, Prof, Mr, Ms) there will be <strong>no</strong> initials (e.g. <em>no</em> Shepherd,John A)

In other words, you can ignore the possibility of certain types of names while implementing your input and output functions.

A more precise definition can be given using a BNF grammar:

<table width="642">

 <tbody>

  <tr>

   <td width="642">PersonName ::= Family’,’Given | Family’, ‘GivenFamily     ::= NameList Given      ::= NameList NameList   ::= Name | Name’ ‘NameListName       ::= Upper Letters Letter     ::= Upper | Lower | PuncLetters    ::= Letter | Letter Letters Upper      ::= ‘A’ | ‘B’ | … | ‘Z’Lower      ::= ‘a’ | ‘b’ | … | ‘z’Punc       ::= ‘-‘ | “‘”</td>

  </tr>

 </tbody>

</table>

You should not make any assumptions about the maximum length of a PersonName.

Under this syntax, the following are valid names:

Smith,John

Smith, John

O’Brien, Patrick Sean

Mahagedara Patabendige,Minosha Mitsuaki Senakasiri

I-Sun, Chen Wang

Clifton-Everest,Charles Edward

The following names are <em>not</em> valid in our system:

Jesus                     # no single-word names

Smith  ,  Harold          # space before the “,”

Gates, William H., III    # no initials, too many commas

A,B C                     # names must at least 2 letters

Smith, john               # names begin with an upper-case letter

Think about why each of the above is invalid in terms of the syntax definition. <strong>Important</strong>: for this assignment, we define an ordering on names as follows:

the ordering is determined initially by the ordering on the Family Name

if the Family Names are equal, then the ordering is determined by the Given Names ordering of parts is determined lexically

There are examples of how this works in the section on Operations on PersonNames below.

<h2>Representing Person Names</h2>

The first thing you need to do is to decide on an internal representation for your PersonName data type. You should do this, however, after you have looked at the description of the operators below, since what they require may affect how you decide to structure your internal PersonName values.

When you read strings representing PersonName values, they are converted into your internal form, stored in the database in this form, and operations on PersonName values are carried out using this data structure. It is useful to define a <em>canonical form</em> for names, which may be slightly different to the form in which they are read (e.g. “Smith, John” might be rendered as “Smith,John”). When you display PersonName values, you should show them in canonical form, regardless of how they were entered or how they are stored.

The first functions you need to write are ones to read and display values of type PersonName. You should write analogues of the functions complex_in(), complex_out that are defined in the file complex.c. Call them, e.g., pname_in() and pname_out(). Make sure that you use the V1 style function interface (as is done in complex.c).

Note that the two input/output functions should be complementary, meaning that any string displayed by the output function must be able to be read using the input function. There is no requirement for you to retain the precise string that was used for input (e.g. you could store the PersonName value internally in a different form such as splitting it into two strings: one for the family name(s), and one for the given name(s)).

One thing that pname_in() must do is determine whether the name has the correct structure (according to the grammar above). Your pname_out() should display each name in a format that can be read by pname_in().

Note that you are <em>not</em> required to define binary input/output functions, called receive_function and send_function in the PostgreSQL documentation, and called complex_send and complex_recv in the complex.cfile.

As noted above, you cannot assume anything about the maximum length of names. If your solution uses two fixed-size buffers (one for family, one for given) then your mark is limited to 6/10.

<h2>Operations on person names</h2>

You must implement all of the following operations for the PersonName type:

<strong><em> PersonName</em></strong><strong><em><sub>1</sub> = PersonName</em></strong><strong><em><sub>2</sub></em></strong> … two names are equal

Two PersonNames are equivalent if, they have the same family name(s) and the same given name(s).

PersonName<sub>1</sub>: Smith,John

PersonName<sub>2</sub>: Smith, John

PersonName<sub>3</sub>: Smith, John David

PersonName<sub>4</sub>: Smith, James




(PersonName<sub>1</sub> = PersonName<sub>1</sub>) is true

(PersonName<sub>1</sub> = PersonName<sub>2</sub>) is true

(PersonName<sub>2</sub> = PersonName<sub>1</sub>) is true        <em>(commutative)</em>

(PersonName<sub>2</sub> = PersonName<sub>3</sub>) is false

(PersonName<sub>2</sub> = PersonName<sub>4</sub>) is false

<strong><em> PersonName</em></strong><strong><em><sub>1</sub> &gt; PersonName</em></strong><strong><em><sub>2</sub></em></strong> … the first PersonName is greater than the second

<em>PersonName</em><em><sub>1</sub></em> is greater than <em>PersonName</em><em><sub>2</sub></em> if the Family part of <em>PersonName</em><em><sub>1</sub></em> is lexically greater than the Family part of <em>PersonName</em><em><sub>2</sub></em>. If the Family parts are equal, then <em>PersonName</em><em><sub>1</sub></em> is greater than <em>PersonName</em><em><sub>2</sub></em> if the Given part of <em>PersonName</em><em><sub>1</sub></em> is lexically greater than the Given part of <em>PersonName</em><em><sub>2</sub></em>.

PersonName<sub>1</sub>: Smith,James

PersonName<sub>2</sub>: Smith,John

PersonName<sub>3</sub>: Smith,John David

PersonName<sub>4</sub>: Zimmerman, Trent




(PersonName<sub>1</sub> &gt; PersonName<sub>2</sub>) is false

(PersonName<sub>1</sub> &gt; PersonName<sub>3</sub>) is false (PersonName<sub>3</sub> &gt; PersonName<sub>2</sub>) is true

(PersonName<sub>1</sub> &gt; PersonName<sub>1</sub>) is false

(PersonName<sub>4</sub> &gt; PersonName<sub>3</sub>) is true

Other operations:   <strong>&lt;&gt;</strong>,   <strong>&gt;=</strong>,   <strong>&lt;</strong>,   <strong>&lt;=</strong>

You should also implement the above operations, whose semantics is hopefully obvious from the descriptions above. The operators can typically be implemented quite simply in terms of the first two operators.

<strong> family(<em>PersonName</em>)</strong> returns just the Family part of a name

<table width="603">

 <tbody>

  <tr>

   <td width="603">PersonName<sub>1</sub>: Smith,JamesPersonName<sub>2</sub>: O’Brien,Patrick SeanPersonName<sub>3</sub>: Mahagedara Patabendige,Minosha Mitsuaki SenakasirPersonName<sub>4</sub>: Clifton-Everest,David Ewan family(PersonName<sub>1</sub>) returns “Smith” family(PersonName<sub>2</sub>) returns “O’Brien”family(PersonName<sub>3</sub>) returns “Mahagedara Patabendige” family(PersonName<sub>4</sub>) returns “Clifton-Everest”</td>

  </tr>

 </tbody>

</table>

<strong> given(<em>PersonName</em>)</strong> returns just the Given part of a name

<table width="603">

 <tbody>

  <tr>

   <td width="603">PersonName<sub>1</sub>: Smith,JamesPersonName<sub>2</sub>: O’Brien,Patrick SeanPersonName<sub>3</sub>: Mahagedara Patabendige,Minosha Mitsuaki SenakasirPersonName<sub>4</sub>: Clifton-Everest,David Ewan given(PersonName<sub>1</sub>) returns “James” given(PersonName<sub>2</sub>) returns “Patrick Sean” given(PersonName<sub>3</sub>) returns “Minosha Mitsuaki Senakasir” given(PersonName<sub>4</sub>) returns “David Ewan”</td>

  </tr>

 </tbody>

</table>

<strong> show(<em>PersonName</em>)</strong> returns a displayable version of the name

It appends the entire Family name to the first Given name (everything before the first space, if any), separated by a single space.

<table width="603">

 <tbody>

  <tr>

   <td width="603">PersonName<sub>1</sub>: Smith,JamesPersonName<sub>2</sub>: O’Brien,Patrick SeanPersonName<sub>3</sub>: Mahagedara Patabendige,Minosha Mitsuaki SenakasirPersonName<sub>4</sub>: Clifton-Everest,David EwanPersonName<sub>5</sub>: Bronte,Greta-Anna Maryanneshow(PersonName<sub>1</sub>) returns “James Smith” show(PersonName<sub>2</sub>) returns “Patrick O’Brien” show(PersonName<sub>3</sub>) returns “Minosha Mahagedara Patabendige” show(PersonName<sub>4</sub>) returns “David Clifton-Everest” show(PersonName<sub>5</sub>) returns “Greta-Anna Bronte”</td>

  </tr>

 </tbody>

</table>

<strong>Hint:</strong> test out as many of your C functions as you can <em>outside</em> PostgreSQL (e.g. write a simple test driver) before you try to install them in PostgreSQL. This will make debugging much easier.

You should ensure that your definitions <em>capture the full semantics of the operators</em> (e.g. specify commutativity if the operator is commutative). You should also ensure that you provide sufficient definitions so that users of the PersonName type can create <strong>hash-based</strong> indexes on an attribute of type PersonName.