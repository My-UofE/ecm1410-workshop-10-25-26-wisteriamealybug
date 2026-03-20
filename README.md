[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/ykcm0c8p)
[![Open in Codespaces](https://classroom.github.com/assets/launch-codespace-2972f46106e565e64193e422d61a12cf1da4916b45550586e14ef0a7c637dd04.svg)](https://classroom.github.com/open-in-codespaces?assignment_repo_id=23211950)
## MessageBoard java project (with post tags and reply functionality)

In this workshop we are going to create a messageboard application in which users can add and view posts. This is similar to the tasks in Workshop 9 but have been revised, so please read and follow the updated instructions.

Note for this workshop you start with an empty directory. You can initially keep all files in the working directory. Once you have the code working you can organise it into a strucutred project (see instructions below).

### `Post.java`

We will start by creating a java object class to store the message board posts.

#### TASK 1

Create a file called `Post.java` and add the following code:

```java
import java.io.*;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

public class Post implements Serializable {

    // Attributes
    private static int idCounter = 0;
    private int postID;
    private String author;
    private String subject;
    private String message;
    private String[] tags;
    private int parentID;
    private int date;

    // constructor for posts
    public Post(String author, String subject, String message) {
        this(author, subject, message, new String[0], -1, null);
    }

    // constructor allows posts with custom datestamps (for debugging)
    public Post(String author, String subject, String message, LocalDate date) {
        this(author, subject, message, new String[0], -1, date);
    }

    // constructor that allows additional attributes of tags and parent post ID
    public Post(String author, String subject, String message, String[] tags, int parentID){
        this(author, subject, message, tags, parentID, date);
    }

    // base constructor with arguments for all attributes 
    public Post(String author, String subject, String message, String[] tags, int parentID, LocalDate date) {
        this.postID = ++idCounter;
        this.author = author;
        this.subject = subject;
        this.message = message;
        this.tags = tags;
        this.parentID = parentID;
        if (date == null) {
            this.date = (int)LocalDate.now().toEpochDay();
        } else {
            this.date = (int)date.toEpochDay();
        }
    }

    public String toString() {
        String result = String.format("Post[postID=%d, author=\"%s\", subject=\"%s\", message=\"%s\", date=%d, tags=[], parentID=-1]", 
                                postID, author, subject, message.replace("\n", "\\n"), date);
        return result;
    }

}
```

**NOTES**

1. The static attribute `idCounter` exists to provide a counter to label the posts with an integer identifier. Each time the constructor is called `idCounter++` increments the counter which is used to provide the next `postID` value.

2. For this example project we record the date each post was made. We can create a post without specifying a date, and in this case the current date is used via a call to `LocalDate.now()`. 

Within the `Post` object the date is stored as an `int` epoch day, (epoch dates/times measure time since 1st January 1970). To convert a java `LocalDate` object to epoch day we use method `.toEpochDay()`.

We can convert a date stored as `int` epoch day back to a `LocalDate` using `LocalDate.ofEpochDay()`.

#### TASK 2

Write a test application `TestPostApp.java` which creates a post with information as below:


| | |
|-|-|
| Author: | `Alex Adam` |
| Subject: | `Help with Java` |
|  Message: | `Hi, could anyone help me I need to learn how to code in java!` |

Add code to print the object (using the `toString()` as above). 

As the date in the above posts is unspecified the constructor will use the current date from `LocalDate.now()`.

Look at the printed value of the date (it should be approximately `55*365` days). 

#### `LocalDates`

The following example code shows how we can create a `LocalDate` with a custom date in a variety of ways, and use formatting to customise the way it is displayed. Add the code below to your test app (delete after testing!):

```java
LocalDate date1 = LocalDate.of(2023,3,30); // 30th March 2023
LocalDate date2 = LocalDate.now().minusDays(7); // seven days before today
LocalDate date3 = LocalDate.ofEpochDay(50); // epoch day 50

// create formatter to diplay dates in day/month/year format
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");

System.out.println(date1.format(formatter));
System.out.println(date2.format(formatter));
System.out.println(date3.format(formatter));
```

#### TASK 3

In the `Post.java` file add the following getters:

 `getPostID` `getAuthor` `getSubject` `getMessage` `getDate`

NOTE: the getDate getter can return the stored date as an int 

Also add a method to print a formatted version of the post

```java
    public String toFormattedString() {
        LocalDate postDate = LocalDate.ofEpochDay(this.date);
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yy");
        String result = "\n------------------  Post " + postID + "  -------------------" + 
        "\nAuthor: " + author + 
        "\nDate: " + postDate.format(formatter) + 
        "\nSubject: " + subject + "\n" + 
        "----  Message:  -------------------------------\n" +  
        message + 
        "\n-----------------------------------------------\n";
        return result;
    }
```

Create a method that can save the formatted post as a text file (not serialising it!). Hint refer to the slides from Week 8. 

Check you can open and view the resulting file.

Hint. To write a string to file we use code like:

```java
BufferedWriter out = new BufferedWriter(new FileWriter(filename));
out.write( "Hello World" );
out.close();
```

Your method should throw any IOException that results from the operation i.e.

```java
public void saveAsTextFile(String filename) throws IOException {
    
}
```

Test by adding the following in `TestPostApp.java`, saving the example post to file `mypost.txt` using a `try` and `catch` as indicated below:

```java
try {
        // code to save the post
} catch( IOException ex ) {
        System.out.println("File not saved.");
        ex.printStackTrace();
}
```

### `MessageBoardInterface.java`

The following file defines an interface for a MessageBoard class. Save this into file `MessageBoardInterface.java`

```java
import java.io.*;
import java.time.LocalDate;

/**
 * The MessageBoardInterface provides methods for managing and interacting with a message board.
 * It allows adding, deleting, searching, displaying, and backing up posts.
 */
public interface MessageBoardInterface extends Serializable {

    /**
     * Retrieves an array of all post IDs on the message board.
     *
     * @return an array of integers representing the IDs of all posts
     *        on the message board or an empty array if there are no posts.
     */
    public int[] getPostIDs();

    /**
     * Adds a post to the message board with the specified author, subject, and message.
     * The current date will be used as the date of the post.
     *
     * @param author  the author of the post (must not be null or empty)
     * @param subject the subject of the post (must not be null or empty)
     * @param message the message content of the post (must not be null or empty)
     * @return the ID of the newly added post
     * @throws IllegalArgumentException if any of the parameters are invalid
     */
    public int addPost(String author, String subject, String message) throws IllegalArgumentException;

    /**
     * Adds a post to the message board with the specified author, subject, message, tags and parent post ID.
     * The current date will be used as the date of the post.
     *
     * @param author  the author of the post (must not be null or empty)
     * @param subject the subject of the post (must not be null or empty)
     * @param message the message content of the post (must not be null or empty)
     * @param tags a string containing comma separated tags to associated with the post (may be null or empty string)
     * @param parentID either the ID of the post which this post is a reply to, or -1 to indicate the post has no parent
     * @return the ID of the newly added post
     * @throws IllegalArgumentException if any of the parameters are invalid
     */
    public int addPostAdvanced(String author, String subject, String message, String tagString, int parentID) throws IllegalArgumentException;


    /**
     * Deletes the post with the specified ID from the message board.
     *
     * @param postID the ID of the post to be deleted
     * @throws IDInvalidException if the post ID is invalid
     */
    public void deletePost(int postID) throws IDInvalidException;

    /**
     * Searches for posts with subjects that contain the specified keyword.
     *
     * @param keyword the keyword to search for in the subjects (case-insensitive)
     * @return an array of post IDs that match the search criteria
     */
    public int[] searchPostsBySubject(String keyword);

    /**
     * Searches for posts within the specified date range.
     *
     * @param startDate the start date of the range (inclusive)
     * @param endDate   the end date of the range (inclusive)
     * @return an array of post IDs that match the search criteria 
     */
    public int[] searchPostsByDate(int startDate, int endDate);

    /**
     * Displays the formatted post with the specified ID.
     *
     * @param postID the ID of the post to be displayed
     * @throws IDInvalidException if the post ID is invalid
     */
    public String getFormattedPost(int postID) throws IDInvalidException;

    /**
     * Backs up the message board to a file.
     *
     * @throws IOException if an I/O error occurs during the backup process
     */
    public void saveMessageBoard(String filename) throws IOException;

    /**
     * Loads the message board state from a backup file.
     * 
     * The current contents of the message board will be replaced by the contents of the backup file.
     *
     * @throws IOException            if an I/O error occurs during the loading process
     * @throws ClassNotFoundException if the class of a serialized object cannot be found
     */
    public void loadMessageBoard(String filename) throws IOException, ClassNotFoundException;

    /**
     * Saves a post as formatted text to a file.
     *
     * @param postID   the ID of the post to save
     * @param filename the name of the file to save the messages to
     * @throws IDInvalidException if the post ID is invalid
     * @throws IOException if an I/O error occurs during the saving process
     */
    public void savePostAsTextFile(int postID, String filename)  throws IDInvalidException, IOException; 

    /**
     * Searches for replies to posts with specified parentID.
     *
     * @param parentID  the parent post to search for children of.
     * @return an array of post IDs that match the criteria 
     */
    public int[] getReplyPosts(int parentID);

    /**
     * Searches for posts with subjects that contain the specified tag.
     *
     * @param tag  the tag to search for in the list of tags (case-insensitive)
     * @return     an array of post IDs that match the search criteria
     */
    public int[] searchPostsByTag(String tag);

    /**
     * Returns list of all tags added to the system, in alphabetical order, without duplicates
     *
     * @return     an array of tag strings
     */
    public String[] getAllTags();


}
```

### `IDInvalidException.java`

The interface includes a custom exception which is thrown when an interface method is called with an invalid `postID` argument.

To set up the custom exception we need to save the following code in file `IDInvalidException.java`:

```java
public class IDInvalidException extends RuntimeException {
    public IDInvalidException(String m) {
        super(m);
    }
}
```

### `MessageBoard.java`

Write a `MessageBoard` class to implement this interface. 

Start with the following code in file `MessageBoard.java` note that we will not add `implements MessageBoardInterface` until we have finished coding all the methods.

A note on exceptions. **The `MessageBoard.java` class is not meant to catch any exceptions** and the methods should throw those as listed in the `MessageBoardInterface.java` (they will get handled by the user interface system). 


```java
import java.util.*;
import java.time.LocalDate;
import java.io.*;
import java.time.format.DateTimeFormatter;


public class MessageBoard implements Serializable {
    private List<Post> posts;
    private String boardName;

    public MessageBoard(String boardName) {
        this.boardName = boardName;
        this.posts = new ArrayList<>();
    }
}
```

You can see that the implementation of the messageboard system has attributes of a board name, and a list of posts that have been added. The constructor for the message board, takes the name and sets up a new empty `ArrayList` to store the posts.

The interface has been designed so that it does not directly deal with the Post objects, but can access them via their `postID` identifiers.

#### TASK 4

Add the following code that lets us access the message board data. As several of the interface methods require us to find a post from its `postID` attribute we have added a helper function that takes `postID` and returns its index in the `posts` `ArrayList`. If the `postID` does not exist it throws an `IDInvalidException`:

```java
    public String getBoardName() {
        return boardName;
    }

    public int[] getPostIDs() {
        int[] postIDs = new int[posts.size()];
        int i = 0;
        for (Post post : posts) {
            postIDs[i++] = post.getPostID();
        }
        return postIDs;
    }

    public int getPostIndex(int postID) throws IDInvalidException {
        for (int i = 0; i < posts.size(); i++) {
            if (posts.get(i).getPostID() == postID) {
                return i;
            }
        }
        throw new IDInvalidException("Invalid post ID.");
    }
```

Now we can edit `MessageBoard.java` to add the `addPost()` and  `getFormattedPost()` interface methods (refer to the interface to see the documentation on what the method should do and return):

```java
public int addPost(String author, String subject, String message){
    // this should create a new post and add it to the posts ArrayList
}
```

```java
public String getFormattedPost(int postID) throws IDInvalidException{
    // this should make use of getPostIndex to access the post
    // and print it using the .toFormattedString() method
}
```

To test your `MessageBoard` code create a file `TestMBApp.java`. 

In the main method of this file create a new `MessageBoard` with board name "Coding Support" and use `addPost()` to post the following messages:

| | |
|-|-|
| Author: | `Alex Adam` |
| Subject: | `Help with Java` |
| Message: | `Hi, could anyone help me I need to learn how to code in java!` |

| | |
|-|-|
| Author; | `Belinda Bennett` |
| Subject: | `Help with Java` |
|  Message: | `Hi Alex. Yes I can send some tutorials I found useful.` |

| | |
|-|-|
| Author: | `Cindy Carter` |
| Subject: | `Coding on a Chromebook` |
|  Message: | `Hi, could anyone help me I need to learn how to code in java!` |

| | |
|-|-|
| Author: | `Dennis Dobson` |
| Subject: | `Windows problems` |
|  Message: | `My windows laptop is stuck on a reboot loop. Does anyone know what to do!` |

To check these have been correctly added use `getPostIDs` to retrieve the posts and print them to screen using `getFormattedPost()`.

Check that your test application compiles and runs as expected.

#### TASK 5

Next add the methods `searchPostsBySubject()` and `deletePost()`. 

Hint. As before while searching you can can store the posts within a `List<Post>`, then loop over this to return a `int[]` containing the `postID` values.

In your test application check your methods, by selecting all posts with a subject matching `java` and deleting the posts from the message board. 

After deleting select and print the board messages to check the posts were deleted correctly.

#### TASK 6

Below your existing code in `TestMBApp.java` Add two new posts but specifiying custom date values. 

To do this you will need to edit `MessageBoard.java` to add a second `addPost` method that can accept a date, e.g. like 

```java
public int addPost(String author, String subject, String message, int epochDate)
```

NOTE: You will need to convert the integer `epochDate` into a `LocalDate` object e.g. using `LocalDate.ofEpochDay( ... )`, so that you can use the following constructor already defined in `Post.java`:

```
public Post(String author, String subject, String message, LocalDate date)
```

When completed write code to add the following posts (with custom date values) in `TestMBApp.java`:

| | |
|-|-|
| Author: | `Ellie` |
| Subject: | `Java IDE` |
| Message: | `Can someone recommend a Java IDE?` |
| Date: | `20148` (as epoch date) |

| | |
|-|-|
| Author: | `Fred Fansha` |
| Subject: | `Java IDE` |
| Message: | `I just use VS Code` |
| Date: | `20149` (as epoch date) |


Add the method `searchPostsByDate()` and in your test application check that with a `startDate` of `20147` and `endDate` of `20149` you can correctly select just these posts using the method you wrote, and display them.


### Adding `serialisable` to save/load board


#### TASK 7 

The message board is required to have the ability to be serialisable, so it can save its contents to disk, and to restore its state from this file.

Note that when the MessageBoard calls the `loadMessageBoard` from the file it should delete all of its existing data and replace it with the loaded data. 

This means that rather than serialising the MessageBoard object, we need to serialise and deserialise its attributes. 

The following shows how we can save the MessageBoard data to file:

```java
public void saveMessageBoard(String filename) throws IOException{
    ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(filename));
    // store boardName attribute
    out.writeObject(boardName);
    // convert posts to array Post[] to simplifies the deserialisation
    Post[] postArray = posts.toArray(new Post[posts.size()]);
    //  store Post array
    out.writeObject(postArray);
} 
```

Write the corresponding `loadMessageBoard()` method which should throw `IOException, ClassNotFoundException` as indicated in the interface.

Hint. Refer to the week 8 slides for how to deserialise. 

You will have to first load the `boardName` and then read in the `Post[]` array.

Check the save method works by saving the MessageBoard to file `codingsupport.ser` at the end of your `TestMessageBoardApp.java`, handling the exception as indicated below

```java
try {
        // code to save the message board
} catch( IOException ex ) {
        System.out.println("Board not saved.");
        ex.printStackTrace();
}
```

Create a second test program called `TestMBLoadApp.java`. This should create a main class that creates a new MessageBoard and uses the load method to restore the posts from the saved file. It should handle the thrown exceptions as indicated below.

```
try {
    // code to load message board
} catch( IOException ex ) {
    System.out.println("Board not loaded.");
    ex.printStackTrace();
} catch( ClassNotFoundException ex ) {
    System.out.println("Could not find class.");
    ex.printStackTrace();
}
```

Check they are loaded correctly by displaying the loaded posts to screen.


#### TASK 8 

Implement the interface method `savePostAsTextFile()`. This should call the `saveAsTextFile()` method from your `Post` class to save a given post.

In your `TestMBLoadApp.java` file add code to find the post with subject including search term `windows`, and to save the first matching post (there should be only one!) as test file `windowspost.txt`. Catch and handle the exceptions as above i.e.

```java
try {
        // code to save the post
} catch( IOException ex ) {
        System.out.println("File not saved.");
        ex.printStackTrace();
}
```

#### TASK 9 (ADDITIONAL REFDEF TASKS)

In the `Post` class this REFDEF assignment has additional attributes `tags` and `parentID`. 

1. Edit the code to add functionality related to these attributes. To do this first add the method `addPostAdvanced()` to `MessageBoard.java`. In addition to `author` `subject` `message` values, this method also takes values for `tagString` and `parentID` arguments.

   - here `tagString` should be provided as a single string with tags separated with commas i.e. "java, IDE, urgent" would resolve to three tags ["java", "IDE", "urgent"]. You should convert the string to an array `String[]` for use with the relevant `Post` constructor method. 
 
   - the `parentID` should either be set to `-1` to indicate the post is not a reply to a previous post, or be set to a valid parent post ID to indicate it is a reply to an earlier post. Your code should check that if `parentID` is not `-1`, then it refers to a valid messageboard post. If an invalid ID has been given it should raise an `IllegalArgumentException`.

2. edit the `toString` method so that the details of the tags and parent ID are correctly displayed (the current code shows these as empty / unspecified). You do not need to change the `toFormattedString` method.

3. complete the additional interface methods so they work in line with the interface documentation:

    ```
    public int[] getReplyPosts(int parentID);

    public int[] searchPostsByTag(String tag);

    public String[] getAllTags();
    ```

4. Create a third test program called `TestMBRefDef.java` that tests the code methods you have written.

5. One this is working edit the `MessageBoard` definition to specify it `implements MessageBoardInterface` and check it compiles without errors.

### `package messageboard;`

#### TASK 10

The final task is to arrange your work into a java project folder structure and prepare a jar fileto distribute your code as a java package `messageboard`.

Clean up your directory by deleting the `.class` files. Then:

 - create folders `src` `bin` `doc` `build` `res` `TestSystem`

 - move test program files into `TestSystem`
 
 - add `package messageboard;` lines at top of messageboard `.java` files
 
 - store `.java` files stored in `messageboard` package directory within `src`
 
 - rebuild the `.class` binaries storing to `bin` folder

```sh
javac -d bin -cp bin src/messageboard/*.java
```

 - create jar file in `build` folder called `mboard_v1.0.jar` containing package binaries.

```sh
jar cvf ./build/mboard_v1.0.jar -C bin .
```

 - you can test your package with your test programs. Note you will have to add line `import messageboard.*;` to the top of your files to import the package.

 ```sh
javac -cp ./build/mboard_v1.0.jar ./TestSystem/*.java
java -cp ./build/mboard_v1.0.jar:TestSystem TestMBApp
java -cp ./build/mboard_v1.0.jar:TestSystem TestMBLoadApp
 ```

Well done! 
