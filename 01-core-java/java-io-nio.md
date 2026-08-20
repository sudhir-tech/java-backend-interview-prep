# Java I/O & NIO — Interview Preparation

A practical guide to Java input/output, file handling, NIO, `Path`, `Files`, streams, buffering, channels, serialization and backend interview scenarios.

---

# 1. What Is Java I/O?

Java I/O provides APIs for reading and writing data.

Common sources and destinations include:

```text
Files
Console
Network connections
Memory
Other streams
```

The classic I/O APIs are mainly under:

```java
java.io
```

Modern file and NIO APIs are mainly under:

```java
java.nio
java.nio.file
```

---

# 2. Input vs Output

### Input

Reading data into the application.

```text
File → Application
Network → Application
Console → Application
```

### Output

Writing data from the application.

```text
Application → File
Application → Network
Application → Console
```

---

# 3. Byte Streams vs Character Streams

Java I/O broadly provides two categories:

```text
Byte streams
Character streams
```

### Byte streams

Used for binary data.

Main base classes:

```java
InputStream
OutputStream
```

Examples:

```java
FileInputStream
FileOutputStream
BufferedInputStream
BufferedOutputStream
```

### Character streams

Designed for character/text data.

Main base classes:

```java
Reader
Writer
```

Examples:

```java
FileReader
FileWriter
BufferedReader
BufferedWriter
```

---

# 4. InputStream

`InputStream` is the base abstraction for reading bytes.

Example:

```java
try (InputStream input =
         new FileInputStream("data.txt")) {

    int value;

    while ((value = input.read()) != -1) {
        System.out.println(value);
    }
}
```

`read()` returns:

```text
0–255 → byte value
-1    → end of stream
```

---

# 5. OutputStream

`OutputStream` is the base abstraction for writing bytes.

Example:

```java
try (OutputStream output =
         new FileOutputStream("data.txt")) {

    output.write(65);
}
```

This writes the byte corresponding to the value `65`.

For text, prefer character-oriented APIs or explicitly specify an encoding.

---

# 6. Reader

`Reader` is the base abstraction for reading character data.

Example:

```java
try (Reader reader =
         new FileReader("data.txt")) {

    int value;

    while ((value = reader.read()) != -1) {
        System.out.print((char) value);
    }
}
```

---

# 7. Writer

`Writer` is the base abstraction for writing character data.

Example:

```java
try (Writer writer =
         new FileWriter("data.txt")) {

    writer.write("Hello Java");
}
```

For production applications, explicitly choosing the character encoding is often preferable.

---

# 8. Byte Stream vs Character Stream

Use byte streams for:

```text
Images
PDFs
ZIP files
Audio
Video
Other binary data
```

Use character streams for:

```text
Text files
CSV
Logs
JSON
XML
Source code
```

Interview answer:

> Byte streams work with raw bytes and are suitable for binary data, while character streams are designed for character data and handle character decoding and encoding.

---

# 9. FileInputStream

Reads raw bytes from a file.

```java
try (FileInputStream input =
         new FileInputStream("input.txt")) {

    byte[] buffer = new byte[1024];

    int count;

    while ((count = input.read(buffer)) != -1) {
        System.out.println(
            new String(buffer, 0, count)
        );
    }
}
```

For text, use an explicit charset instead of relying on the platform default.

---

# 10. FileOutputStream

Writes bytes to a file.

```java
try (FileOutputStream output =
         new FileOutputStream("output.txt")) {

    output.write("Hello".getBytes(
        java.nio.charset.StandardCharsets.UTF_8
    ));
}
```

The second constructor argument can enable append mode:

```java
new FileOutputStream(
    "output.txt",
    true
);
```

---

# 11. BufferedInputStream

Adds buffering around an input stream.

```java
try (BufferedInputStream input =
         new BufferedInputStream(
             new FileInputStream("data.bin")
         )) {

    int value;

    while ((value = input.read()) != -1) {
        // process byte
    }
}
```

Buffering can reduce the number of expensive underlying I/O operations.

---

# 12. BufferedOutputStream

Buffers output operations.

```java
try (BufferedOutputStream output =
         new BufferedOutputStream(
             new FileOutputStream("data.bin")
         )) {

    output.write(data);
}
```

Call:

```java
output.flush();
```

when appropriate to push buffered data to the underlying stream.

Closing the stream also flushes it.

---

# 13. BufferedReader

`BufferedReader` is useful for efficient character reading and line-oriented text processing.

```java
try (BufferedReader reader =
         Files.newBufferedReader(
             Path.of("data.txt")
         )) {

    String line;

    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
}
```

---

# 14. BufferedWriter

`BufferedWriter` provides buffered character output.

```java
try (BufferedWriter writer =
         Files.newBufferedWriter(
             Path.of("data.txt")
         )) {

    writer.write("Hello");
    writer.newLine();
    writer.write("Java");
}
```

---

# 15. What Is Buffering?

A buffer temporarily stores data in memory.

Without buffering:

```text
Application
    ↓
Underlying I/O
    ↓
Repeated operations
```

With buffering:

```text
Application
    ↓
Memory buffer
    ↓
Larger I/O operations
    ↓
Underlying resource
```

This can improve performance by reducing the number of underlying I/O operations.

---

# 16. flush()

`flush()` asks an output stream/writer to push buffered data to the underlying destination.

Example:

```java
writer.write("Important data");
writer.flush();
```

It does not generally mean the data is durably persisted to physical storage.

For stronger durability guarantees, the underlying resource and operating system semantics matter.

---

# 17. close()

`close()` releases a resource.

Examples:

```text
File handle
Socket
Stream
Reader
Writer
```

Closing an output stream typically also flushes pending buffered data.

---

# 18. try-with-resources

Modern Java provides try-with-resources for automatic resource management.

Example:

```java
try (BufferedReader reader =
         Files.newBufferedReader(
             Path.of("data.txt")
         )) {

    String line =
        reader.readLine();
}
```

The resource is automatically closed when the try block finishes.

---

# 19. AutoCloseable

Try-with-resources works with types implementing:

```java
AutoCloseable
```

Many I/O resources implement it.

Example:

```java
class MyResource
        implements AutoCloseable {

    @Override
    public void close() {
        System.out.println("Closed");
    }
}
```

Then:

```java
try (MyResource resource =
         new MyResource()) {

    // use resource
}
```

---

# 20. Why try-with-resources?

It helps prevent resource leaks.

Without it:

```java
Reader reader = null;

try {
    reader = new FileReader("data.txt");
    // process
}
finally {
    if (reader != null) {
        reader.close();
    }
}
```

Try-with-resources is shorter and handles close operations more reliably.

---

# 21. Suppressed Exceptions

If an exception occurs in the try block and another exception occurs while closing a resource, the close exception can become a suppressed exception.

You can inspect them using:

```java
Throwable[] suppressed =
    exception.getSuppressed();
```

This is an advanced interview topic.

---

# 22. File

`java.io.File` represents a file or directory path and provides legacy file-system operations.

Example:

```java
File file =
    new File("data.txt");

System.out.println(file.exists());
```

For modern applications, `Path` and `Files` are generally preferred.

---

# 23. Path

`Path` from NIO represents a path in the file system.

```java
Path path =
    Path.of("data.txt");
```

Useful operations:

```java
path.getFileName();
path.getParent();
path.toAbsolutePath();
```

---

# 24. Paths

The `Paths` utility class was commonly used before newer Java versions introduced convenient `Path.of()` methods.

Example:

```java
Path path =
    Paths.get("data.txt");
```

Modern code can commonly use:

```java
Path path =
    Path.of("data.txt");
```

---

# 25. Files

`java.nio.file.Files` provides utility methods for file-system operations.

Examples:

```java
Files.exists(path);
Files.createFile(path);
Files.delete(path);
Files.copy(source, target);
Files.move(source, target);
```

It also provides convenient reading and writing APIs.

---

# 26. Reading a Small File

For a reasonably small text file:

```java
String content =
    Files.readString(
        Path.of("data.txt")
    );
```

This loads the content into memory, so it should not be used blindly for very large files.

---

# 27. Reading All Lines

```java
List<String> lines =
    Files.readAllLines(
        Path.of("data.txt")
    );
```

Again, this loads the lines into memory.

For large files, prefer streaming or buffered processing.

---

# 28. Writing a String

```java
Files.writeString(
    Path.of("data.txt"),
    "Hello Java"
);
```

You can specify options such as:

```java
StandardOpenOption.CREATE
StandardOpenOption.APPEND
```

---

# 29. Files.lines()

`Files.lines()` provides a lazily populated stream of lines.

Example:

```java
try (Stream<String> lines =
         Files.lines(
             Path.of("data.txt")
         )) {

    lines.filter(line ->
            line.contains("ERROR"))
         .forEach(System.out::println);
}
```

Important:

> The returned stream owns an underlying file resource and should be closed, so use try-with-resources.

---

# 30. Path Operations

Example:

```java
Path base =
    Path.of("/app");

Path file =
    base.resolve("logs/app.log");
```

Other useful methods:

```java
file.normalize();
file.toAbsolutePath();
file.getFileName();
file.getParent();
```

---

# 31. resolve()

`resolve()` combines paths.

```java
Path base =
    Path.of("/app");

Path config =
    base.resolve("config");
```

Conceptually:

```text
/app/config
```

This is preferable to manually concatenating file separators.

---

# 32. normalize()

`normalize()` removes redundant path components such as:

```text
.
..
```

Example:

```java
Path path =
    Path.of(
        "/app/logs/../config"
    );

Path normalized =
    path.normalize();
```

Conceptually:

```text
/app/config
```

Normalization does not necessarily access the file system.

---

# 33. Absolute vs Relative Path

### Relative

```text
config/app.properties
```

Interpreted relative to the current working directory.

### Absolute

```text
/opt/app/config/app.properties
```

Represents a complete path from the file-system root on systems where that concept applies.

---

# 34. Files.exists()

```java
Path path =
    Path.of("data.txt");

if (Files.exists(path)) {
    System.out.println("Exists");
}
```

Be careful with TOCTOU-style logic:

```text
check exists
↓
then use file
```

The file can change between the check and the operation.

When possible, perform the desired operation directly and handle its exception.

---

# 35. Creating Directories

```java
Files.createDirectory(
    Path.of("logs")
);
```

For nested directories:

```java
Files.createDirectories(
    Path.of("app/logs/2026")
);
```

`createDirectories()` creates missing parent directories as needed.

---

# 36. Copying Files

```java
Files.copy(
    source,
    target
);
```

To replace an existing target:

```java
Files.copy(
    source,
    target,
    StandardCopyOption.REPLACE_EXISTING
);
```

---

# 37. Moving Files

```java
Files.move(
    source,
    target,
    StandardCopyOption.REPLACE_EXISTING
);
```

Moving can be useful for:

```text
Archiving
Renaming
Log rotation
File processing workflows
```

Actual atomicity depends on the file system and options used.

---

# 38. Deleting Files

```java
Files.delete(path);
```

or:

```java
Files.deleteIfExists(path);
```

`deleteIfExists()` is useful when absence should not itself be treated as an error.

---

# 39. Listing a Directory

```java
try (Stream<Path> paths =
         Files.list(
             Path.of("logs")
         )) {

    paths.forEach(System.out::println);
}
```

The stream should be closed because it is backed by an open directory resource.

---

# 40. Walking a Directory Tree

```java
try (Stream<Path> paths =
         Files.walk(
             Path.of("logs")
         )) {

    paths.filter(Files::isRegularFile)
         .forEach(System.out::println);
}
```

`Files.walk()` can traverse nested directories.

Because it uses resources, use try-with-resources.

---

# 41. File Attributes

Useful methods include:

```java
Files.size(path);
Files.getLastModifiedTime(path);
Files.isDirectory(path);
Files.isRegularFile(path);
Files.isReadable(path);
Files.isWritable(path);
```

Example:

```java
long size =
    Files.size(path);
```

---

# 42. Charset

Character data must be encoded and decoded.

A charset maps:

```text
Characters ↔ Bytes
```

Common charset:

```text
UTF-8
```

Use:

```java
StandardCharsets.UTF_8
```

instead of depending on the machine's default charset when a stable encoding is required.

---

# 43. Reading with Explicit Charset

```java
try (BufferedReader reader =
         Files.newBufferedReader(
             Path.of("data.txt"),
             StandardCharsets.UTF_8
         )) {

    String line;

    while ((line =
            reader.readLine()) != null) {

        System.out.println(line);
    }
}
```

---

# 44. Writing with Explicit Charset

```java
try (BufferedWriter writer =
         Files.newBufferedWriter(
             Path.of("data.txt"),
             StandardCharsets.UTF_8
         )) {

    writer.write("Hello");
}
```

Explicit encoding avoids platform-dependent behavior.

---

# 45. Serialization

Java serialization converts an object graph into a byte representation.

A traditional serializable class implements:

```java
Serializable
```

Example:

```java
class User
        implements Serializable {

    private String name;
}
```

Then an `ObjectOutputStream` can serialize it.

---

# 46. ObjectOutputStream

Example:

```java
try (ObjectOutputStream output =
         new ObjectOutputStream(
             new FileOutputStream("user.ser")
         )) {

    output.writeObject(user);
}
```

---

# 47. ObjectInputStream

Deserialization:

```java
try (ObjectInputStream input =
         new ObjectInputStream(
             new FileInputStream("user.ser")
         )) {

    User user =
        (User) input.readObject();
}
```

Traditional Java native serialization has security, compatibility and maintenance concerns and should not be used casually for untrusted data.

---

# 48. serialVersionUID

A serializable class can define:

```java
private static final long
    serialVersionUID = 1L;
```

It is used during serialization compatibility checks.

Example:

```java
class User
        implements Serializable {

    private static final long
        serialVersionUID = 1L;

    private String name;
}
```

---

# 49. transient

A `transient` field is skipped by default Java serialization.

Example:

```java
class User
        implements Serializable {

    private String username;

    private transient String password;
}
```

The password field is not serialized by default.

Important:

> `transient` is about Java serialization; it does not automatically mean a field is excluded from every persistence or JSON serialization framework.

---

# 50. Why Native Java Serialization Is Often Avoided

Traditional Java serialization can introduce:

```text
Security risks
Versioning complexity
Large serialized representations
Tight coupling to Java class structure
Difficult interoperability
```

For backend APIs, formats such as:

```text
JSON
Protocol Buffers
Avro
```

are often more appropriate depending on requirements.

---

# 51. NIO

NIO stands for:

```text
New I/O
```

It introduced APIs for more flexible and scalable I/O operations.

Important concepts include:

```text
Path
Files
Channels
Buffers
Selectors
```

For everyday file operations, `Path` and `Files` are especially important.

---

# 52. Channel

A channel provides a connection to an I/O source or destination.

Examples:

```java
FileChannel
SocketChannel
ServerSocketChannel
```

Compared with traditional streams, channels can support operations such as:

```text
Reading/writing through ByteBuffer
Positioning
File transfer
Potentially non-blocking I/O for appropriate channel types
```

---

# 53. ByteBuffer

A `ByteBuffer` stores bytes for channel operations.

Basic flow:

```java
ByteBuffer buffer =
    ByteBuffer.allocate(1024);
```

Then:

```text
write data into buffer
       ↓
flip()
       ↓
read from buffer
       ↓
clear()/compact()
```

---

# 54. ByteBuffer flip()

After writing data into a buffer, call:

```java
buffer.flip();
```

`flip()` prepares the buffer for reading by:

```text
limit = current position
position = 0
```

This is a common NIO interview question.

---

# 55. ByteBuffer clear()

```java
buffer.clear();
```

Prepares the buffer for another write operation.

Conceptually:

```text
position = 0
limit = capacity
```

It does not erase the underlying bytes.

---

# 56. ByteBuffer compact()

```java
buffer.compact();
```

Moves remaining unread data to the beginning and prepares the buffer to receive more data.

This is useful when a buffer contains partially processed data.

---

# 57. FileChannel

`FileChannel` provides channel-based file I/O.

Example:

```java
try (FileChannel channel =
         FileChannel.open(
             Path.of("data.txt"),
             StandardOpenOption.READ
         )) {

    ByteBuffer buffer =
        ByteBuffer.allocate(1024);

    while (channel.read(buffer) != -1) {

        buffer.flip();

        while (buffer.hasRemaining()) {
            System.out.print(
                (char) buffer.get()
            );
        }

        buffer.clear();
    }
}
```

---

# 58. Channel vs Stream

### Stream

```text
InputStream
OutputStream
Reader
Writer
```

Designed around sequential data transfer.

### Channel

```text
FileChannel
SocketChannel
```

Can support richer operations such as:

```text
Positioning
Transfer operations
ByteBuffer-based I/O
Non-blocking modes for applicable channels
```

---

# 59. Selectors

A `Selector` allows a thread to monitor multiple selectable channels for I/O readiness.

Commonly associated with:

```text
SocketChannel
ServerSocketChannel
```

This can support event-driven network servers where one thread handles many connections.

---

# 60. Blocking vs Non-Blocking I/O

### Blocking

A thread can wait until an I/O operation completes.

```text
Thread
  ↓
waits for I/O
```

### Non-blocking

The operation can return without waiting for the complete operation to finish.

```text
Thread
  ↓
initiates/checks I/O
  ↓
continues handling other work
```

NIO networking APIs support non-blocking patterns for appropriate channel types.

---

# 61. NIO.2

Java 7 introduced significant file-system APIs often called NIO.2.

Important APIs:

```text
Path
Files
FileSystem
WatchService
```

These APIs are commonly used instead of legacy `java.io.File`.

---

# 62. WatchService

`WatchService` can monitor directories for file-system events.

Typical events:

```text
ENTRY_CREATE
ENTRY_MODIFY
ENTRY_DELETE
```

Example use cases:

```text
Configuration reload
File ingestion
Log processing
Directory monitoring
```

---

# 63. File Locking

Java provides file locking through channels.

Example:

```java
try (FileChannel channel =
         FileChannel.open(
             Path.of("data.lock"),
             StandardOpenOption.CREATE,
             StandardOpenOption.WRITE
         )) {

    try (FileLock lock =
             channel.lock()) {

        // protected file operation
    }
}
```

The exact behavior of file locks depends on the operating system and file system.

---

# 64. Memory-Mapped Files

`FileChannel` supports memory-mapped file access.

Conceptually:

```text
File
 ↓
Memory mapping
 ↓
MappedByteBuffer
```

This can be useful for large files and certain high-performance workloads, but it requires careful consideration of memory usage and access patterns.

---

# 65. I/O and Backend Applications

Java I/O appears in backend systems for:

```text
File uploads
CSV imports
Log processing
Report generation
Configuration files
Image processing
Batch jobs
Document generation
Data exports
```

---

# 66. File Upload Example

A backend endpoint may receive an uploaded file.

Conceptually:

```text
Client
  ↓
HTTP multipart request
  ↓
Controller
  ↓
Validation
  ↓
Storage service
  ↓
File system / Object storage
```

For production systems, large uploads are often better handled through object storage rather than keeping everything on local application disks.

---

# 67. Large File Processing

Avoid:

```java
String content =
    Files.readString(path);
```

for very large files.

Instead process incrementally:

```java
try (BufferedReader reader =
         Files.newBufferedReader(path)) {

    String line;

    while ((line =
            reader.readLine()) != null) {

        process(line);
    }
}
```

This keeps memory usage more predictable.

---

# 68. Streaming Large Files

For large files:

```text
Read chunk
   ↓
Process chunk
   ↓
Release/reuse buffer
   ↓
Read next chunk
```

rather than:

```text
Read entire file
   ↓
Store entire file in memory
   ↓
Process
```

This is especially important for backend services processing large datasets.

---

# 69. I/O Exception Handling

Many I/O operations throw:

```java
IOException
```

Example:

```java
try {
    String content =
        Files.readString(path);
}
catch (IOException e) {
    // handle or propagate
}
```

Don't simply swallow the exception:

```java
catch (IOException e) {
}
```

Log, wrap, propagate or otherwise handle it according to the application's error strategy.

---

# 70. IOException vs FileNotFoundException

`FileNotFoundException` is a subclass of:

```text
IOException
```

It can occur when opening a file fails, including cases such as a missing file or an inaccessible path.

---

# 71. Resource Leak

A resource leak occurs when an application fails to release a resource.

Examples:

```text
File handle
Socket
Database connection
Stream
Reader
Writer
```

For Java I/O:

```text
try-with-resources
```

is one of the primary tools for preventing leaks.

---

# 72. Encoding Problem

Suppose one application writes:

```text
UTF-8
```

and another reads using an incompatible charset.

The text can become corrupted.

Therefore, for systems that exchange text:

```text
Choose an explicit charset
```

when the encoding is part of the contract.

---

# 73. File Security

Never blindly trust user-provided file paths.

Potential problem:

```text
../../../../etc/passwd
```

This is a path traversal attack.

A secure application should validate and constrain paths before accessing files.

---

# 74. Safe File Storage Concept

Instead of directly using:

```java
Path.of(userProvidedFilename)
```

consider:

```text
Validate input
      ↓
Generate safe server-side name
      ↓
Resolve against allowed directory
      ↓
Normalize
      ↓
Verify path remains within allowed root
      ↓
Perform operation
```

File-system security belongs to the application's threat model and should be designed deliberately.

---

# 75. Temp Files

Java provides temporary file utilities.

Example:

```java
Path temp =
    Files.createTempFile(
        "upload-",
        ".tmp"
    );
```

Temporary directories:

```java
Path dir =
    Files.createTempDirectory(
        "processing-"
    );
```

Use secure lifecycle management and clean up temporary data when appropriate.

---

# 76. Files.isSameFile()

Java can compare whether two paths refer to the same file:

```java
Files.isSameFile(path1, path2);
```

This can involve file-system access and should not be confused with simple path-string equality.

---

# 77. File Attributes

For more detailed metadata, Java provides attribute APIs.

Examples:

```text
BasicFileAttributes
PosixFileAttributes
DosFileAttributes
```

Example:

```java
BasicFileAttributes attrs =
    Files.readAttributes(
        path,
        BasicFileAttributes.class
    );
```

---

# 78. Symbolic Links

A symbolic link points to another file-system path.

Java's NIO APIs provide operations for symbolic links.

Be careful when processing untrusted paths because symbolic links can affect where a path actually resolves.

---

# 79. DirectoryStream

`DirectoryStream` provides another way to iterate over directory entries.

Example:

```java
try (DirectoryStream<Path> stream =
         Files.newDirectoryStream(
             Path.of("logs")
         )) {

    for (Path path : stream) {
        System.out.println(path);
    }
}
```

It is useful when you want directory iteration without creating a large list of all entries.

---

# 80. Files.walk() vs Files.list()

### Files.list()

Lists immediate children:

```text
directory
 ├── a.txt
 ├── b.txt
 └── logs/
```

It does not recursively traverse nested directories.

### Files.walk()

Traverses the directory tree recursively.

Use:

```java
Files.walk(path)
```

when recursive traversal is required.

---

# 81. InputStream vs Reader

### InputStream

Works with bytes:

```java
InputStream
```

### Reader

Works with characters:

```java
Reader
```

If you need to decode UTF-8 bytes into characters, a character reader or explicit decoder/charset should be used.

---

# 82. OutputStream vs Writer

### OutputStream

Writes bytes.

### Writer

Writes characters.

For text:

```text
Writer
```

is usually more convenient.

For binary:

```text
OutputStream
```

is appropriate.

---

# 83. FileReader Caveat

`FileReader` is convenient, but code that depends on a specific encoding should use APIs that let you explicitly provide the charset.

Prefer:

```java
Files.newBufferedReader(
    path,
    StandardCharsets.UTF_8
);
```

when UTF-8 is the required format.

---

# 84. Files.readString() vs BufferedReader

### readString()

Good for reasonably small files:

```java
String data =
    Files.readString(path);
```

### BufferedReader

Better when processing line by line:

```java
try (BufferedReader reader =
         Files.newBufferedReader(path)) {

    // process lines
}
```

The choice depends on file size and processing requirements.

---

# 85. I/O Performance

Performance depends on:

```text
File size
Buffer size
Storage system
Network latency
Access pattern
Number of I/O operations
Encoding/decoding
Concurrency
```

Avoid optimizing blindly.

Measure the actual bottleneck.

---

# 86. I/O and Threads

Blocking I/O can occupy a thread while waiting for the underlying resource.

In high-concurrency applications, this matters because:

```text
More blocked threads
        ↓
More memory/context-switch overhead
```

Modern backend frameworks can use different concurrency models, including asynchronous or non-blocking approaches where appropriate.

---

# 87. Virtual Threads and I/O

Modern Java provides virtual threads.

They are particularly useful for workloads with many blocking operations because a large number of concurrent tasks can be represented without requiring one expensive platform thread per task.

Conceptually:

```text
Virtual thread
      ↓
blocking I/O
      ↓
JVM can suspend/unmount task
      ↓
platform thread can execute other work
```

Virtual threads do not make CPU-bound work faster by themselves.

---

# 88. I/O vs CPU-Bound Work

### I/O-bound

The application spends significant time waiting for:

```text
Database
Network
Disk
External API
```

### CPU-bound

The application spends significant time computing:

```text
Compression
Encryption
Large calculations
Image processing
```

I/O concurrency strategies and CPU parallelism strategies are not identical.

---

# 89. Backend Interview Scenario

### Question

You need to process a 10 GB CSV file. How would you approach it?

### Answer

> I would avoid loading the entire file into memory. I'd process it incrementally using a buffered reader or streaming approach, validate each record, process it in manageable batches and handle failures according to the business requirements. If processing is expensive, I would also consider controlled concurrency and backpressure.

---

# 90. Backend Interview Scenario

### Question

How would you read a large log file and find ERROR lines?

### Answer

```java
try (Stream<String> lines =
         Files.lines(path)) {

    lines.filter(line ->
            line.contains("ERROR"))
         .forEach(System.out::println);
}
```

For more complex processing, a buffered reader can provide explicit line-by-line control.

---

# 91. Backend Interview Scenario

### Question

Why would you use NIO instead of `File`?

### Answer

> NIO provides a more modern and flexible file-system API through `Path` and `Files`, with better support for file attributes, directory traversal, channels, file-system events and other advanced operations.

---

# 92. Backend Interview Scenario

### Question

How do you prevent file descriptor leaks?

### Answer

> I use try-with-resources for streams, readers, writers, channels and other `AutoCloseable` resources. This makes cleanup automatic even when an exception occurs.

---

# 93. Backend Interview Scenario

### Question

How would you safely handle uploaded files?

### Answer

> I would validate the file type and size, avoid trusting the client-provided filename, generate a safe storage name, constrain the destination directory, validate the resolved path, and preferably store large production files in controlled object storage rather than directly on the application server.

---

# 94. Backend Interview Scenario

### Question

Why should you specify UTF-8 explicitly?

### Answer

> Because relying on the platform default charset can produce different behavior across environments. If UTF-8 is part of the application's data contract, I prefer to specify it explicitly.

---

# 95. Backend Interview Scenario

### Question

What is the difference between `Files.readAllLines()` and `Files.lines()`?

### Answer

> `readAllLines()` loads all lines into a collection, so memory usage grows with the file size. `Files.lines()` returns a lazily populated stream and is more suitable for processing large files incrementally. The stream should be closed with try-with-resources.

---

# 96. Backend Interview Scenario

### Question

What is the difference between `Files.list()` and `Files.walk()`?

### Answer

> `Files.list()` processes the immediate entries of a directory, while `Files.walk()` recursively traverses the directory tree. Both return streams that should be closed.

---

# 97. Backend Interview Scenario

### Question

What is `ByteBuffer.flip()`?

### Answer

> After writing data into a ByteBuffer, `flip()` changes the buffer from write mode to read mode by setting the limit to the current position and resetting the position to zero.

---

# 98. Backend Interview Scenario

### Question

What is a channel?

### Answer

> A channel is an abstraction for I/O operations such as file and network communication. Channels work with buffers and can support operations such as positioning and transfers; certain network channels can also operate in non-blocking mode.

---

# 99. Backend Interview Scenario

### Question

What is try-with-resources?

### Answer

> It is a Java language feature that automatically closes resources implementing `AutoCloseable` after the try block finishes, including when an exception occurs. It is the preferred approach for managing most I/O resources.

---

# 100. I/O Quick Revision

```text
InputStream
    ↓
Read bytes

OutputStream
    ↓
Write bytes

Reader
    ↓
Read characters

Writer
    ↓
Write characters

Buffered streams
    ↓
Reduce underlying I/O operations

Path
    ↓
Modern file-system path abstraction

Files
    ↓
Modern file-system operations

try-with-resources
    ↓
Automatic resource cleanup

Charset
    ↓
Character ↔ byte encoding

FileChannel
    ↓
Channel-based file I/O

ByteBuffer
    ↓
Buffer for channel operations

Selector
    ↓
Monitor multiple selectable channels

WatchService
    ↓
Monitor directory changes

Serializable
    ↓
Traditional Java object serialization

transient
    ↓
Exclude field from default Java serialization
```

---

# 101. Most Important Interview Questions

Be comfortable answering:

1. What is Java I/O?
2. Byte stream vs character stream?
3. InputStream vs Reader?
4. OutputStream vs Writer?
5. What is buffering?
6. What does `flush()` do?
7. What does `close()` do?
8. What is try-with-resources?
9. What is AutoCloseable?
10. What is `Path`?
11. `File` vs `Path`?
12. What is `Files`?
13. `Files.readAllLines()` vs `Files.lines()`?
14. `Files.list()` vs `Files.walk()`?
15. Why specify UTF-8?
16. What is NIO?
17. What is a Channel?
18. What is ByteBuffer?
19. What does `flip()` do?
20. What does `clear()` do?
21. What is a Selector?
22. What is WatchService?
23. What is Java serialization?
24. What is serialVersionUID?
25. What is transient?
26. Why avoid native Java serialization for untrusted data?
27. How do you process a huge file?
28. How do you prevent resource leaks?
29. How do you secure file uploads?
30. Blocking vs non-blocking I/O?
31. I/O-bound vs CPU-bound?
32. How do virtual threads relate to blocking I/O?

---

# 102. Final Interview Rule

For Java backend interviews, don't focus only on memorizing API names.

Be able to explain:

```text
What the API does
        ↓
Why you would use it
        ↓
What happens with large data
        ↓
How resources are managed
        ↓
What can go wrong
        ↓
How you would design it in production
```

For example, don't just say:

> "I know Files.lines()."

A stronger answer is:

> "`Files.lines()` gives me lazy line-by-line processing, which can be useful for large text files because I don't need to load the entire file into memory. Since the stream owns an underlying file resource, I use it inside try-with-resources."

That is the level expected from a Java Backend Developer.
