# Lecture 13. JDBC Database API

<h3>Abstract</h3>

<p>This lecture presents the programmers' interfaces defined by the SQL
standard.  We will show a very popular database API called JDBC.  JDBC is
part of the Java environment and will be discussed in more detail in the
lectures on Java.  In this lecture we will compare ADO and JDBC.  They are
similar.  It is not surprising, as the are both derived from the call
level interface of SQL (SQL/CLI).</p>

<p>We will also mention the methods to access a database that are available in the 
scripts written in PHP.</p>

<hr><h3><a name="Interfejsy">APIs of SQL</a></h3>

<ol>
<li><i>modules</i>,
<li><i>embedded SQL</i>,
<li><i>call level interface</i> (e.g. ADO and ADO.NET described by
	<a href="../wyklad12/index.html" target="_new">the previous lecture</a>),
<li><i>direct SQL</i> (described by
	<a href="../wyklad09/index.html" target="_new">lecture 9</a>)
</ol>

<hr><h3><a name="Jez">Module language</a></h3>

<p>Every external program that wants to access an SQL database makes use of a number 
of <i>modules</i>. Each module is a set of procedures.</p>

<p>Each <i>procedure</i> is a set of definitions of parameters and a single SQL
statement that references these parameters, e.g.</p>

<pre>PROCEDURE Delete_Emp (SQLSTATE, :p_Empno NUMBER(4));
  DELETE FROM Emp WHERE Emp.Empno = :p_Empno;</pre>

<p>Each parameter is preceded by a colon, unless it is a standard parameter
<code>SQLSTATE</code> or <code>SQLCODE</code>.</p>

<h4>Parameter <code>SQLSTATE</code></h4>

<p>The <code>SQLSTATE</code> parameter  is used to pass the exit status of the SQL statement
to the application that called the procedure.</p>

<p><code>SQLSTATE</code> equal to <code>00000</code> means that the procedure has been
successfully completed.</p>

<h4>A call to module procedure in PL/I</h4>

<p>The module language allows writing programs in host languages without the need to introduce
any changes in their syntax and semantics, e.g.</p> 

<pre>retcode CHAR(5);
h_Empno NUMBER(4);
...
CALL Delete_Emp(retcode, h_Empno);
IF retcode = '00000' 
THEN ...;		/* successful deletion */
ELSE ...;		/* an error occurred */</pre>


<h3><hr><a name="Osadzony SQL">Embedded SQL</a></h3>

<p>SQL statements can be embedded into the code of an application.
It is a much easier and more natural method than to write modules.
The variables of the host language
that are used inside the embedded SQL statements are preceded by a colon just like
the parameters of module procedures, e.g.</p>  

<pre>EXEC SQL BEGIN DECLARE SECTION;
  	SQLSTATE CHAR(5);
  h_Empno NUMBER(4);
EXEC SQL END DECLARE SECTION;
...
EXEC SQL DELETE FROM Emp WHERE Emp.Empno = :h_Empno;
...
IF retcode = '00000' 
THEN ...;		/* successful deletion */
ELSE ...;		/* an error occurred */</pre>

<p>The statements of the embedded SQL are preceded by <code>EXEC SQL</code>.
Therefore it is easy to find them in the code written in the host language.
These statements are usually followed by a semicolon.</p>

<p>All variables used inside the embedded SQL statements must be declared in
the <code>DECLARE SECTION</code> that is surrounded by commands
<code>EXEC SQL BEGIN DECLARE SECTION</code> and 
<code>EXEC SQL END DECLARE SECTION</code>.</p>

<p>After an embedded SQL statement is executed the program is notified on the exit
status of this operation by means of the <code>SQLCODE</code> and/or
<code>SQLSTATE</code> variables.

<hr><h3><a name="JDBC">JDBC as an instance of Call Level Interface</a></h3>

<p>There are two call level interfaces for Java.</p>

<dl>
<dt><b>SQLJ</b>
<dd>SQL embedded in Java.
<dt><b>JDBC</b>
<dd>Database application programmer's interface for Java.
</dl>

<p>JDBC has become more popular and is used in:</p>

<ol>
<li>Java applications on the client side,
<li>Java applets (small programs) run by web browsers,
<li>Java servlets (programs that response to users' requests) run by web servers,
<li>Java procedures and functions stored in databases.
</ol>

<p><a name="jdbc-def"/><i>JDBC</i> is the collection of Java classes and Java interfaces that offer
universal access to databases in Java code. These classed and interfaces are grouped
together into the <code>java.sql</code> package.  Therefore at the beginning of most
programs that use JDBC there is the the following <code>import</code> statement:</p>

<pre>import java.sql.*;</pre>

<p>The database to be contacted is identified by a URL (= Uniform Resource Locator) of 
the following form (<code>jdbc</code> is the name of the main protocol):</p>

<pre>jdbc:&lt;<i>subprotocol</i>&gt;:&lt;<i>subname</i>&gt;</pre>

<p>For example:</p>

<pre>jdbc:odbc:my_database
jdbc:oracle:thin:@xeon.pjwstk.edu.pl:1521:ORCL</pre>  

<hr><h3><a name="Sterownik JDBC">JDBC driver</a></h3>

<p><i>A JDBC driver</i> is a collection of classes that implement interfaces of 
the <code>java.sql</code> package. In order to connect to a database, you have to
perform two steps.</p>

<ol>
<li>Load the JDBC driver into the Java virtual machine
	<ul>
	<li>by means of the Java mechanism of dynamic class loading, e.g.
	<pre>Class.forName("sun.jdbc.odbc.JdbcOdbcDriver");</pre>
	<li>or by means of the static <code>registerDriver</code> method
		of the
		<code>DriverManager</code> class, e.g.
	<pre>DriverManager.registerDriver(new sun.jdbc.odbc.JdbcOdbcDriver());</pre>
	</ul>
<li>Open connection to the database by means of the 
	<code>getConnection</code> method of  the <code>DriverManager</code>
	class, e.g.

<pre>String URL = "jdbc:odbc:my_database"
Connection con = DriverManager.getConnection(URL, "username", "password");</pre>

</ol>	

<hr><h3><a name="Obiekty">JDBC objects</a></h3>

<p>Here is the description of how to use fundamental classes of JDBC.</p>

<ol>
<li><a name="conn-def"/>A database session starts from the creation of an object of
	the
	<code>Connection</code> class (it corresponds to the <code>Connection</code>
	class
	of ADO).
<li>An object of the <code>Statement</code> class represents the SQL statement
	to be executed (it corresponds to the <code>Command</code> class
	of ADO). Its fundamental methods are:
	<dl>
	<dt><code>executeQuery</code>
	<dd>It takes the text of an SQL query and returns the <i>ResultSet</i>
		object with
		data.
	<dt><code>executeUpdate</code>
	<dd>It takes the text of an SQL statement and returns the number of rows affected 
		by this statement.
	</dl>
<li>An object of the <code>ResultSet</code> class represents the set of records returned
	by the query.  Exactly one record is available at one time. 
	(this class corresponds to the <code>Recordset</code> class 
	of ADO).
	<ul>
	<li>The applications gets subsequent records by calling the
		<code>next</code> method. This method will return <code>false</code> if 
		there is no next record. The return value <code>true</code> means
		that the operation has been successful.
	<li>The values of fields can be read by calling methods like
		<code>getString</code> and <code>getFloat</code>.
		These methods get the name of a column and return the value
		of this column in the current record. The type of this value
		is specified by the name of the method.
	</ul>
<li>An object of the <code>SQLException</code> class represents an error
	(exception) caused by the execution of an SQL statement by the database.
</ol>

<p>Here is an example of an application of JDBC.  We connect to a database 
defined by ODBC name <code>oras</code>. We use username <code>scott</code> and
password <code>tiger</code>.  We assume that <code>scott</code> owns the
<code>Emp</code> table which has a text column <code>Ename</code>
and a numeric column <code>Sal</code>.</p>


<pre>// load the driver
try {
  Class.forName("sun.jdbc.odbc.JdbcOdbcDriver");
}
catch(ClassNotFoundException ex) {
  System.out.println(ex.getMessage ());
}  
 
try {
  //connect to the database

  Connection con = DriverManager.getConnection
                   (&quot;jdbc:odbc:oras&quot;, &quot;scott&quot;, &quot;tiger&quot;);
      
  // create the SQL statement and execute it
  Statement stmt = con.createStatement(); 
  String query = "SELECT Ename, Sal FROM Emp";
  ResultSet rs = stmt.executeQuery(query);
 
  // walk through all returned records
  while (rs.next()) {
    String s = rs.getString("Ename");
    float z = rs.getFloat("Sal");
    System.out.println(s + ":  " + z);
  }
} catch(SQLException ex) {
  System.out.println(ex.getMessage ()
    + ex.getSQLState () + ex.getErrorCode ());
}</pre>
  
<p>In this example we used the driver that is the bridge between ODBC and
JDBC. We could also have used another driver possibly developed by another vendor,
e.g. an Oracle driver.  In this case the URL could be for example
<code>jdbc:oracle:thin:@xeon.pjwstk.edu.pl:1521:ORCL</code>.  As an <a
href="javascript:popUp('ok01.html',600,680)">exercise</a> please test the
enclosed Java program, that contains the code shown above tailored for MS
Access.  </p>

<hr><h3><a name="Modyfikowanie danych">Updating the database</a></h3>

<pre>Statement stm = con.createStatement();
int recs = stm.executeUpdate("UPDATE Emp SET Sal = Sal*1.1");
System.out.println(recs + " employees has got 10% raise.")
stm.close();</pre>

<p>The <code>stm.executeUpdate()</code> call returns a single number.  This is the number
of rows that have been affected by the statement (i.e. that have been updated).
The <code>executeUpdate</code> method can be used to perform operations 
<code>INSERT</code>, <code>UPDATE</code>, <code>DELETE</code> and DDL operations 
like <code>CREATE TABLE</code>, <code>ALTER TABLE</code>, <code>DROP TABLE</code>.</p>

<h4><a name="Modyfikowanie danych2">Update through a <code>ResultSet</code></h4>

<pre>Statement stm = con.createStatement(
                       ResultSet.TYPE_SCROLL_SENSITIVE,
                       ResultSet.CONCUR_UPDATABLE);
String sql = "SELECT * FROM Emp";
ResultSet rs = stm.executeQuery(sql);

rs.first();
rs.updateString("Ename","Meyer");
rs.updateFloat("Sal",10000);
rs.updateRow();</pre>

<p>There are database systems that do not implement updateable <code>ResultSet</code>s.
The same remark has been made about ADO's updateable <code>Recordset</code>s.</p>


<h4>Deletion of the current record</h4>

<pre>rs.deleteRow();</pre>

<h4>Addition of a new record</h4>

<pre>rs.moveToInsertRow();
rs.updateString("Ename","Sawyer");
rs.updateFloat("Sal", 10000);
rs.insertRow();</pre>

<hr><h3><a name="Obsluga transakcji">Transactions</a></h3>

<p>By default each SQL statement executed by JDBC is a separate transaction
and is automatically committed (<i>auto-commit</i>).  You can change it by
means of the <code>setAutoCommit</code> method of object
<code>Connection</code>.  Then, when all operations of your transaction are
successful, you will have to commit it explicitly by the <code>commit</code>
method of object <code>Connection</code>.  If an error occurs
during the transaction, you can abort it by means of method
<code>rollback</code> of object <code>Connection</code>. For example:</p>

<pre>try {
   con.setAutoCommit(false);
   Statement stm = con.createStatement();
   stm.executeUpdate(" .....");
   stm.executeUpdate(" .....");
   .....
   con.commit();
}
catch(SQLException e){
   con.rollback();
}</pre>

<hr><h3><a name="Obsluga NULL">Handling <code>NULL</code></a></h3>

<p>Call the <code>wasNull()</code> method if you want to check whether the retrieved column
holds the <code>NULL</code> value.</p> 

<pre>float Sal = rs.getFloat("Sal");
if (rs.wasNull(){
   Sal = 0;
}</pre>
   
<hr><h3><a name="Przygo">Prepared statements</code></a></h3>

<p>If you call the same statement many times, it will be worth to prepare it
(parse it) earlier and then execute it multiple times.  Furthermore, such a
statement can have parameters that are changed at each execution. If you
want to take advantage of these features, use class
<code>PreparedStatement</code> instead of <code>Statement</code>. Its formal
parameters are indicated by question marks, e.g.:</p>
 

<pre>PreparedStatement stmt = 
      con.prepareStatement("UPDATE Emp SET Deptno = ? " 
                        + " WHERE Deptno = ?");</pre>

<p>Methods <code>setInt()</code>, <code>setString()</code>, <code>setDate()</code> etc. are
used to set actual values for the formal parameters of the prepared statement.
Each of them has two arguments: the position of a parameter and its value.
After you prepare the statement, you can set its parameters to a different value each time.
Here is an example code that moves all employees from department 10 to department 50.</p>

<pre>PreparedStatement stmt = 
      con.prepareStatement("UPDATE Emp SET Deptno = ? " 
                        + " WHERE Deptno = ?");
stmt.setInt(1, 50);
stmt.setInt(2, 10);
System.out.println(stmt.executeUpdate()+" moved.");
stmt.close();</pre>
  
<hr><h3><a name="PHP">A remark on PHP</a></h3>

<p>PHP is a web script language with very simple and intuitive syntax 
similar to Java, C and Perl.  It is very popular, because it has a lot of libraries that
facilitate accessing any database server. Furthermore, PHP is free and open source.
It a phenomenon similar to Linux.</p>

<p>PHP has a separate access library for every database management system. Here we present
the library for Oracle database.</p>

<p>In PHP scriptlets (fragments of code) are surrounded by tags <code>&lt;?php</code> 
and <code>?&gt;</code> or simply <code>&lt;?</code> and <code>?&gt;</code>.</p>

<p>For example:</p>

<pre>&lt;?php
  echo "text to be presented";
  print("text to be presented');
?&gt;</pre>

<p>This command adds text to the generated HTML document.  It will be sent to the browser
and displayed on the user's screen.  Names of variables are preceded by the dollar
symbol. If we want to display the value of a variable, we can type:</p>

<pre>&lt;?
  $var = "PHP";
  echo "The subject of this point is ".$var
?&gt;</pre>
  
<p>In order to connect to an Oracle database, we use the
<code>OCILogon</code> function.  This functions returns the object that represents
the connection to the database.</p>
 
<pre>$con = OCILogon("scott", "tiger", "myDb");</pre>

<p>When the connection is no longer needed, we shall close it.</p>

<pre>OCILogoff($con);</pre>

<p>To execute a statement, we create an object that represents it.
We can obtain it as a result of the <code>OCIParse</code> function of the object
that represents the database connection.</p>

<pre>$stmt = OCIParse($con, "DELETE FROM Dept WHERE Deptno = 40");</pre>

<p>When the statement is created, we can execute it.</p>

<pre>OCIExecute($stmt);</pre>

<p>If the executed statement is SELECT, we can browse its results by means of 
the <code>OCIFetchInto</code> function. It pushes the cursor one position forward and returns
<code>true</code> if and only if there exists the next row.  Otherwise, this function
returns <code>false</code>.</p>

<pre>&lt;?php
  $conn = OCILogon("scott", "tiger", "myDb");
  if ($conn == false) {
    die("Error in OCILogon");
  }
  $query = &quot;SELECT Ename, Job, Sal FROM Emp&quot;;
  $stmt = OCIParse($conn, $query);
  if(!$stmt) { // in case of a parse error
    $oerr = OCIError($stmt);
    echo $oerr["message"];
  } else {
    OCIExecute($stmt);
    echo "&lt;TABLE&gt;&quot;;
    while (OCIFetchInto($stmt, $row, OCI_ASSOC))) {
      echo "&lt;TR&gt;&lt;TD&gt;$row[ENAME]&lt;/TD&gt;&quot;
      echo &quot;&lt;TD&gt;$row[JOB]&lt;/TD&gt; &lt;TD&gt;$row[SAL]&lt;/TD&gt;&lt;/TR&gt;";
    }
    echo "&lt;/TABLE>";
  }
?&gt;</pre>

<p>Notice the convenient way to index array <code>$row</code>
that contains the result row. We use just the name of columns, e.g.  
<code>$row[ENAME]</code>.</p>
  
<hr><h3><a name="Podsumowanie">Summary</a></h3>

<p>Lectures 12 and 13 have been devoted to the call level interface, i.e.
the method to call SQL statements in conventional programming languages.
We have compared two approaches: ADO and JDBC. In both interfaces we start from
initiating database connection by means of an object of class <code>Connection</code>.
Then we execute SQL statements by calling appropriate methods of the connection object
or methods of the dedicated object that represents statements
(<code>Command</code> in VBA and <code>Statement</code> in Java).
In the case of a SELECT statement the resulting rows are stored and processed by methods
of an object of class <code>Recordset</code> (VBA) or <code>ResultSet</code> (Java).</p>

<p>In PHP we contact an Oracle database by means of the following functions.</p>

<dl>
<dt><code>OCILogon</code>
<dd>Connect to the database.
<dt><code>OCIParse</code>
<dd>Prepare the SQL statement.
<dt><code>OCIExecute</code>
<dd>Execute the prepared SQL statement.
<dt><code>OCIFetchInto</code>
<dd>Retrieve the next resulting row (for SELECT statements only).
</dl>

<hr><h3><a name="Slownik">Dictionary</a></h3>

<dl>

<dt><a href="#conn-def">Connection</a>
<dd>The class of objects that represent connections to databases.
<dt><a href="#jdbc-def">JDBC</a>
<dd>The collection of Java classes and Java interfaces that offer universal
	access to databases in Java code. These classed and interfaces are
	grouped together into <code>package java.sql</code>. 
<dt><a href="#Sterownik JDBC">JDBC driver</a>
<dd>A JDBC driver is a collection of classes that implement interfaces
	of package <code>package java.sql</code>.
<dt><a href="#Przygo">prepared statement</a>
<dd>The SQL statement that is prepared in advance
	(parsed) and then executed many times. Furthermore, it can have parameters.
	The values of these parameters can altered at each execution. 

<dt><a href="#Obiekty">ResultSet</a>
<dd>The class of objects that represent sets of records returned by queries.

<dt><a href="#Obiekty">Statement</a>
<dd>The class of objects that represent SQL statements.

</dl>
