## Why Use Databases

In the past, our primary way of saving data to the disk was to serialize it into a large JSON-encoded string and write it out to a file. This works well for small amounts of data, such as config or game save files, because it is easy to implement, and the resulting files are relatively easy for a human to read and modify. There is, however, a downside. JSON files can't be read or updated incrementally; you must read and parse the entire file in memory, and overwrite the entire file when you want to update it.

For larger data sets, it becomes important to incrementally read and update it. This is what databases provide us with. We'll be focusing in particular on relational databases, which organize data into tables with rows and columns (conceptually similar to Excel spreadsheets). To interact with it, we'll be using a language called SQL (Structured Query Language), developed by IBM in the 70s.

## Basic SQL Commands and Data Types

There are only five basic commands you need to know to be productive with SQL. You need to be able to create a table, and then you need to know how to create, read, update, and delete records in that table. The commands look like this:

* `CREATE TABLE`
* `INSERT`
* `SELECT`
* `UPDATE`
* `DELETE`

Let's start with creating a table. Doing so requires us to specify all the columns we want in the table. Every column needs a name and a type. This is similar to defining fields in a Java class. For example, let's say we want to make a table that stores the players in a game. We want to keep track of the the player's name, whether or not they are alive, their score, and their health. In Java, we would use `String`, `boolean`, `int`, and `double` for these respectively. In SQL, we'll use these types:

* `VARCHAR`
* `BOOLEAN`
* `INT`
* `DOUBLE`

With this in mind, we can write all five commands to create the table and create/read/update/delete a player in the table. Keep in mind that capitalization isn't important, but generally built-in keywords are written in all caps and table/column names are in snake case (lower case with underscores).

* `CREATE TABLE players (name VARCHAR, is_alive BOOLEAN, score INT, health DOUBLE)`
* `INSERT INTO players VALUES ('Bob', true, 0, 100.0)`
* `SELECT * FROM players WHERE name = 'Bob'`
* `UPDATE players SET is_alive = FALSE WHERE name = 'Bob'`
* `DELETE FROM players WHERE name = 'Bob'`

## HelloDatabase

Create a new project called `HelloDatabase`. In `File -> Project Structure -> Libraries`, add the following library from Maven:

* `com.h2database:h2:1.4.191`

H2 is an embedded SQL database, which means it runs inside your project and stores its data in a single file on your disk. Later on we will try using an external database like PostGreSQL, which will run outside of our project, but for now we can stick to H2 so we don't need to worry about starting and stopping our database. Another nice thing about H2 is that it has a built-in web interface for interacting with the database. In your main class, write just a single line to start it:

```java
public class Main {
    public static void main(String[] args) throws SQLException {
        Server.createWebServer().start();
    }
}
```

After running it, open your web browser and go to http://localhost:8082 where you will see a login page. Change the JDBC URL to `jdbc:h2:./main`, which will save the database in a file called `main.mv.db` in your project's directory. Make sure the username and password are blank and click Connect.

![](https://raw.githubusercontent.com/oakes/java-assignments/master/curriculum/assets/sql-basics-1.png)

An interface will appear that allows you to run SQL commands on the database. Let's try running all five commands, but with a twist: We're going to add an `id` column which assigns a unique number to each record starting at 1. Nearly every database table uses this as a simple way to reference a record, so there is a special type called `IDENTITY` in H2 which will automatically increment it for you. Let's start with the `CREATE TABLE` statement:

![](https://raw.githubusercontent.com/oakes/java-assignments/master/curriculum/assets/sql-basics-2.png)

Now let's do the next four commands. Note that when inserting, we are writing `NULL` for the `id` column so the database generates it for us automatically:

![](https://raw.githubusercontent.com/oakes/java-assignments/master/curriculum/assets/sql-basics-3.png)
![](https://raw.githubusercontent.com/oakes/java-assignments/master/curriculum/assets/sql-basics-4.png)
![](https://raw.githubusercontent.com/oakes/java-assignments/master/curriculum/assets/sql-basics-5.png)
![](https://raw.githubusercontent.com/oakes/java-assignments/master/curriculum/assets/sql-basics-6.png)

This interface is a nice way to interact with your database, but we we need to learn how to run these queries directly from our Java code. Go back to your main class and create a database connect to the aforementioned JDBC URL:

```java
public class Main {
    public static void main(String[] args) throws SQLException {
        Server.createWebServer().start();
        
        Connection conn = DriverManager.getConnection("jdbc:h2:./main");
    }
}
```

This is the connection we will use to send SQL commands to H2. To do so, we will create a `Statement` object and using it to execute SQL strings. We'll start by creating our table and inserting a record. Note that the normal `CREATE TABLE` command will fail if the table already exists, so we must call `CREATE TABLE IF NOT EXISTS` instead:

```java
public class Main {
    public static void main(String[] args) throws SQLException {
        Server.createWebServer().start();
        
        Connection conn = DriverManager.getConnection("jdbc:h2:./main");
        Statement stmt = conn.createStatement();
        stmt.execute("CREATE TABLE IF NOT EXISTS players (id IDENTITY, name VARCHAR, is_alive BOOLEAN, score INT, health DOUBLE)");
        stmt.execute("INSERT INTO players VALUES (NULL, 'Alice', true, 0, 100.0)");
    }
}
```

Restart your project and log back into the web interface. If you run `SELECT * FROM PLAYERS` you should see Alice in the table of results. Now add the `UPDATE` and `DELETE` commands:

```java
public class Main {
    public static void main(String[] args) throws SQLException {
        ...
        stmt.execute("UPDATE players SET is_alive = FALSE WHERE name = 'Alice'");
        stmt.execute("DELETE FROM players WHERE name = 'Alice'");
    }
}
```

When you restart your project and run the same command in the web interface, you should now see no results.

While this works, most of the time we are not hard-coding values into our SQL queries like this. Typically, we will receive data from an external source (such as an HTTP route) and put the values into our SQL string. You may be tempted to reach for `String.format`, which we've used in the past to inject things into strings, but this is a very bad idea. To see why, let's try it out.

```java
public class Main {
    public static void main(String[] args) throws SQLException {
        ...
        
        // BAD
        String name = "Charlie";
        stmt.execute(String.format("INSERT INTO players VALUES (NULL, '%s', true, 0, 100.0)", name));
    }
}
```

Why is this bad? Imagine the string is coming from a web form. This string could contain anything. What if they type in `O'Brien`? The SQL command will be `INSERT INTO players VALUES (NULL, 'O'Brien', true, 0, 100.0)`. Notice the problem? The apostrophe will close the string early, and the query will fail. If the user is clever enough, they might type in something like `', true, 0, 0); DROP TABLE players; --` which would cause your code to delete the entire table (try it!). This is called a SQL injection attack.

The solution is to use a `PreparedStatement` instead of a `Statement` whenever you need to inject values into your SQL query. Let's write an example that inserts the data safely:

```java
public class Main {
    public static void main(String[] args) throws SQLException {
        ...
        
        // GOOD
        PreparedStatement stmt2 = conn.prepareStatement("INSERT INTO players VALUES (NULL, ?, ?, ?, ?)");
        stmt2.setString(1, "David");
        stmt2.setBoolean(2, true);
        stmt2.setInt(3, 0);
        stmt2.setDouble(4, 100.0);
        stmt2.execute();
    }
}
```

We now need to read from the database programmatically. This time, I just want to select all the players and print them out. In this case, we don't actually need to inject anything into the query, because it is simply `SELECT * FROM players`, but we'll use a `PreparedStatement` anyway just to make it a habit:

```java
public class Main {
    public static void main(String[] args) throws SQLException {
        ...
        
        PreparedStatement stmt3 = conn.prepareStatement("SELECT * FROM players");
        ResultSet results = stmt3.executeQuery();
        while (results.next()) {
            String playerName = results.getString("name");
            double health = results.getDouble("health");
            int score = results.getInt("score");
            System.out.printf("%s %s %s\n", playerName, health, score);
        }
    }
}
```

## ToDo

Open the `ToDo` project we made in the past. We're going to add a database to it. Start by adding `com.h2database:h2:1.4.191` to the project. At the top of the main method, create the table:

```java
public class ToDo {
    public static void main(String[] args) throws SQLException {
        Server.createWebServer().start();
        Connection conn = DriverManager.getConnection("jdbc:h2:./main");
        Statement stmt = conn.createStatement();
        stmt.execute("CREATE TABLE IF NOT EXISTS todos (id IDENTITY, text VARCHAR, is_done BOOLEAN)");
        
        ...
    }
}
```

Next we'll create a method to insert a todo item:

```java
public class ToDo {
    public static void insertToDo(Connection conn, String text) throws SQLException {
        PreparedStatement stmt = conn.prepareStatement("INSERT INTO todos VALUES (NULL, ?, false)");
        stmt.setString(1, text);
        stmt.execute();
    }

    ...
}
```

We'll also need a method that returns all the todo items from the database:

```java
public class ToDo {
    ...
    
    public static ArrayList<ToDoItem> selectToDos(Connection conn) throws SQLException {
        ArrayList<ToDoItem> items = new ArrayList<>();
        Statement stmt = conn.createStatement();
        ResultSet results = stmt.executeQuery("SELECT * FROM todos");
        while (results.next()) {
            int id = results.getInt("id");
            String text = results.getString("text");
            boolean isDone = results.getBoolean("is_done");
            items.add(new ToDoItem(id, text, isDone));
        }
        return items;
    }

    ...
}
```

Finally, we need a method to toggle the `is_done` value:

```java
public class ToDo {
    ...
    
    public static void toggleToDo(Connection conn, int id) throws SQLException {
        PreparedStatement stmt = conn.prepareStatement("UPDATE todos SET is_done = NOT is_done WHERE id = ?");
        stmt.setInt(1, id);
        stmt.execute();
    }
    
    ...
}
```

Let's put them to use. In your main method, remove the `ArrayList<ToDoItem> todos` that currently stores them in memory.
