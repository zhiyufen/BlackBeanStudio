# DocumentsProvider基础学习笔记

[TOC]

## 存储访问框架 (SAF)

**Storage Access Framework(SAF)**：存储访问框架，在`Android4.4（API 级别 19）`引入的一个新玩意，为用户浏览手机中的 存储内容提供了便利，可供访问的内容不仅包括：文档，图片，视频，音频，下载，而且包含所有由 由特定`ContentProvider（须具有约定的API: DocumentsProvider）`提供的内容。不管这些内容来自于哪里，不管是哪个应用调用浏览系统文件内容的命令，系统都会用一个统一的界面让你去浏览。
该界面是一个内置的应用程序，叫做DocumentsUI，因为它的IntentFilter不带有LAUNCHER，所以不会在桌面上显示；

云存储服务或本地存储服务可实现用于封装其服务的 `DocumentsProvider`，从而加入此生态系统。客户端应用如需访问提供程序中的文档，只需几行代码即可与 SAF 集成。

SAF 包含以下元素：

- **文档提供程序** - 一种内容提供程序，可让存储服务（如 Google 云端硬盘）提供其管理的文件。文档提供程序以 `DocumentsProvider` 类的子类形式实现。文档提供程序的架构基于传统的文件层次结构，但其实际的数据存储方式由您决定。
- **客户端应用** - 一种定制化的应用，它会调用 `ACTION_CREATE_DOCUMENT`、`ACTION_OPEN_DOCUMENT` 和 `ACTION_OPEN_DOCUMENT_TREE` intent 操作并接收文档提供程序返回的文件。
- **选择器** - 一种系统界面，可让用户访问所有文档提供程序内满足客户端应用搜索条件的文档。

以下为 SAF 提供的部分功能：

- 让用户浏览所有文档提供程序的内容，而不仅仅是单个应用的内容。
- 让您的应用获得对文档提供程序所拥有文档的长期、持续访问权限。用户可通过此访问权限添加、修改、保存和删除提供程序中的文件。
- 支持多个用户帐号和临时根目录，如只有在插入 U 盘后才会出现的“USB 存储提供程序”。

### 数据模型

SAF 的核心是一个内容提供程序，它是 `DocumentsProvider` 类的一个子类。在文档提供程序内，数据结构采用传统的文件层次结构：

![数据模型](https://developer.android.com/static/images/providers/storage_datamodel.png?hl=zh-cn)

**图 1.** 文档提供程序数据模型。根目录指向单个文档，然后整个结构树以此为基点向外延伸。

- 每个文档提供程序都会报告一个或多个根目录（文档树结构的起点）。每个根目录都有唯一的 `COLUMN_ROOT_ID`，并且指向一个表示该根目录下内容的文档（目录）。根目录采用动态设计，以支持多个帐号、临时 USB 存储设备或用户登录/退出等用例。
- 每个根目录下都只有一个文档。该文档指向 1 到 N 个文档，其中每个文档又可指向 1 至 N 个文档。
- 每个存储后端都会使用唯一的 `COLUMN_DOCUMENT_ID` 来引用各个文件和目录，以便将其呈现出来。文档 ID 必须具有唯一性，且一经发出便不得更改，因为这些 ID 会用于实现 URI 持久授权（不受设备重启影响）。
- 文档可以是可打开的文件（具有特定的 MIME 类型），也可以是包含其他文档的目录（具有 `MIME_TYPE_DIR` MIME 类型）。
- 每个文档可拥有不同的功能，具体在 `COLUMN_FLAGS` 中指定。例如，`FLAG_SUPPORTS_WRITE`、`FLAG_SUPPORTS_DELETE` 和 `FLAG_SUPPORTS_THUMBNAIL`。多个目录中可包含相同的 `COLUMN_DOCUMENT_ID`。

### 控制流

文档提供程序数据模型基于传统的文件层次结构。不过，只要能通过 `DocumentsProvider` API 访问数据，您实际上就可以采用自己喜欢的任何方式来存储数据。


![应用](https://developer.android.com/static/images/providers/storage_dataflow.png?hl=zh-cn)

**图 2.** 存储访问框架流

- 在 SAF 中，提供程序和客户端并不直接交互。客户端会请求与文件进行交互（即读取、修改、创建或删除文件）的权限。
- 当应用（在本示例中为照片应用）触发 `ACTION_OPEN_DOCUMENT` 或 `ACTION_CREATE_DOCUMENT` intent 后，交互便会开始。intent 可包含过滤器，用于进一步细化条件，例如“为我提供所有 MIME 类型为‘图片’”的可打开文件”。
- 当 intent 触发后，系统选择器会联络每个已注册的提供程序，并向用户显示匹配内容的根目录。
- 选择器会为用户提供标准的文档访问界面，即使底层的文档提供程序可能相互之间差异很大，一致性也不受影响。例如，图 2 展示了一个 Google 云端硬盘提供程序、一个 USB 提供程序和一个云提供程序。

### 客户端访问

在 Android 4.3 及更低版本中，如果您想让应用从其他应用中检索文件，则该应用必须调用 `ACTION_PICK` 或 `ACTION_GET_CONTENT` 等 intent。然后，用户必须选择一个要从中选取文件的应用，并且所选应用必须提供用户界面，以便用户浏览和选择可用文件。

在 Android 4.4（API 级别 19）及更高版本中，您还可选择使用 `ACTION_OPEN_DOCUMENT` intent，此 intent 会显示由系统控制的选择器界面，以便用户浏览其他应用提供的所有文件。借助此界面，用户便可从任何受支持的应用中选择文件。

在 Android 5.0（API 级别 21）及更高版本中，您还可以使用 `ACTION_OPEN_DOCUMENT_TREE` intent，借助此 intent，用户可以选择供客户端应用访问的目录。

启动DocumentsUi, 浏览图片:

```java
Intent intent = new Intent(Intent.ACTION_OPEN_DOCUMENT);
intent.addCategory(Intent.CATEGORY_OPENABLE);
intent.setType("image/*");
startActivity(intent);
```

>  **注**：`ACTION_OPEN_DOCUMENT` 并非用于替代 `ACTION_GET_CONTENT`。您应使用的 intent 取决于应用的需要：
>
> - 如果您只想让应用读取/导入数据，请使用 `ACTION_GET_CONTENT`。使用此方法时，应用会导入数据（如图片文件）的副本。
> - 如果您想让应用获得对文档提供程序所拥有文档的长期、持续访问权限，请使用 `ACTION_OPEN_DOCUMENT`。例如，照片编辑应用可让用户编辑存储在文档提供程序中的图片。

如需详细了解如何使用系统选择器界面支持浏览文件和目录，请参阅有关如何[访问文档和其他文件](https://developer.android.com/training/data-storage/shared/documents-files?hl=zh-cn)的指南。

## 自定义文档提供程序

### 声明自定义的DocumentsProvider(Manifest)

```xml
<manifest... >
        ...
        <uses-sdk
            android:minSdkVersion="19"
            android:targetSdkVersion="19" />
            ....
            <provider
                android:name="com.example.android.storageprovider.MyCloudProvider"
                android:authorities="com.example.android.storageprovider.documents"
                android:grantUriPermissions="true"
                android:exported="true"
                android:permission="android.permission.MANAGE_DOCUMENTS">
                <intent-filter>
                    <action android:name="android.content.action.DOCUMENTS_PROVIDER" />
                </intent-filter>
            </provider>
        </application>

</manifest>
```

- 设置为 `"true"` 的属性 `android:grantUriPermissions`。此设置允许系统授权其他应用访问您的提供程序中的内容。
- `MANAGE_DOCUMENTS` 权限。默认情况下，提供程序面向所有人提供。添加此权限会将您的提供程序限制于系统。此限制对于确保安全性至关重要。
- 包含 `android.content.action.DOCUMENTS_PROVIDER` 操作的 Intent 过滤器，以便您的提供程序在系统搜索提供程序时出现在选择器中。

### 支持搭载 Android 4.3 及更低版本的设备

详细请参考：https://developer.android.com/guide/topics/providers/create-document-provider?hl=zh-cn#43

### Contracts(合同)

`Contracts`类是 `public final` 类，其中包含 URI、列名称、MIME 类型以及适用于提供程序的其他元数据的常量定义。SAF 会为您提供这些合同类，因此您无需自行编写：

- [`DocumentsContract.Document`](https://developer.android.com/reference/android/provider/DocumentsContract.Document)
- [`DocumentsContract.Root`](https://developer.android.com/reference/android/provider/DocumentsContract.Root)

例如，当用户通过文档提供程序查询文档或根目录时，以下是您可能在光标中返回的列：

```kotlin
    private val DEFAULT_ROOT_PROJECTION: Array<String> = arrayOf(
            DocumentsContract.Root.COLUMN_ROOT_ID,
            DocumentsContract.Root.COLUMN_MIME_TYPES,
            DocumentsContract.Root.COLUMN_FLAGS,
            DocumentsContract.Root.COLUMN_ICON,
            DocumentsContract.Root.COLUMN_TITLE,
            DocumentsContract.Root.COLUMN_SUMMARY,
            DocumentsContract.Root.COLUMN_DOCUMENT_ID,
            DocumentsContract.Root.COLUMN_AVAILABLE_BYTES
    )
    private val DEFAULT_DOCUMENT_PROJECTION: Array<String> = arrayOf(
            DocumentsContract.Document.COLUMN_DOCUMENT_ID,
            DocumentsContract.Document.COLUMN_MIME_TYPE,
            DocumentsContract.Document.COLUMN_DISPLAY_NAME,
            DocumentsContract.Document.COLUMN_LAST_MODIFIED,
            DocumentsContract.Document.COLUMN_FLAGS,
            DocumentsContract.Document.COLUMN_SIZE
    )
    
```

根目录的光标需要包含某些必需列。 这些列是：

- `COLUMN_ROOT_ID`
- `COLUMN_ICON`
- `COLUMN_TITLE`
- `COLUMN_FLAGS`
- `COLUMN_DOCUMENT_ID`
- `COLUMN_AVAILABLE_BYTES`

文档的光标需要包含以下必需列：

- `COLUMN_DOCUMENT_ID`
- `COLUMN_DISPLAY_NAME`
- `COLUMN_MIME_TYPE`
- `COLUMN_FLAGS`
- `COLUMN_SIZE`
- `COLUMN_LAST_MODIFIED`

### 创建 DocumentsProvider 的子类

编写自定义文档提供程序的下一步是子类化抽象类 `DocumentsProvider`。您必须至少实现以下方法：

- `queryRoots()`
- `queryChildDocuments()`
- `queryDocument()`
- `openDocument()`

上述是您必须严格实现的全部方法，还有很多您可能想要实现的方法。如需了解详情，请参阅 [DocumentsProvider](https://developer.android.com/reference/android/provider/DocumentsProvider?hl=zh-cn)。

#### 定义根目录

在实现 `queryRoots()` 时，您需要使用 `DocumentsContract.Root` 中定义的列，返回指向文档提供程序的所有根目录的 `Cursor`。

在以下代码段中，`projection` 参数表示调用方想要返回的特定字段。该代码段会创建一个新光标并向其中添加一行 - 一个根目录（一个顶级目录，例如“下载内容”或“图片”）。大部分提供程序只有一个根目录。您可能会有多个根目录，例如，如果有多个用户帐号，便有多个根目录。在这种情况下，只需向光标再添加一行即可。

```kotlin
    override fun queryRoots(projection: Array<out String>?): Cursor {
        // Use a MatrixCursor to build a cursor
        // with either the requested fields, or the default
        // projection if "projection" is null.
        val result = MatrixCursor(resolveRootProjection(projection))

        // If user is not logged in, return an empty root cursor.  This removes our
        // provider from the list entirely.
        if (!isUserLoggedIn()) {
            return result
        }

        // It's possible to have multiple roots (e.g. for multiple accounts in the
        // same app) -- just add multiple cursor rows.
        result.newRow().apply {
            add(DocumentsContract.Root.COLUMN_ROOT_ID, ROOT)

            // You can provide an optional summary, which helps distinguish roots
            // with the same title. You can also use this field for displaying an
            // user account name.
            add(DocumentsContract.Root.COLUMN_SUMMARY, context.getString(R.string.root_summary))

            // FLAG_SUPPORTS_CREATE means at least one directory under the root supports
            // creating documents. FLAG_SUPPORTS_RECENTS means your application's most
            // recently used documents will show up in the "Recents" category.
            // FLAG_SUPPORTS_SEARCH allows users to search all documents the application
            // shares.
            add(
                DocumentsContract.Root.COLUMN_FLAGS,
                DocumentsContract.Root.FLAG_SUPPORTS_CREATE or
                    DocumentsContract.Root.FLAG_SUPPORTS_RECENTS or
                    DocumentsContract.Root.FLAG_SUPPORTS_SEARCH
            )

            // COLUMN_TITLE is the root title (e.g. Gallery, Drive).
            add(DocumentsContract.Root.COLUMN_TITLE, context.getString(R.string.title))

            // This document id cannot change after it's shared.
            add(DocumentsContract.Root.COLUMN_DOCUMENT_ID, getDocIdForFile(baseDir))

            // The child MIME types are used to filter the roots and only present to the
            // user those roots that contain the desired type somewhere in their file hierarchy.
            add(DocumentsContract.Root.COLUMN_MIME_TYPES, getChildMimeTypes(baseDir))
            add(DocumentsContract.Root.COLUMN_AVAILABLE_BYTES, baseDir.freeSpace)
            add(DocumentsContract.Root.COLUMN_ICON, R.drawable.ic_launcher)
        }

        return result
    }

    private static String[] resolveRootProjection(String[] projection) {
        return projection != null ? projection : DEFAULT_ROOT_PROJECTION;
    }
    private static final String[] DEFAULT_ROOT_PROJECTION = new String[]{
            Root.COLUMN_ROOT_ID,
            Root.COLUMN_FLAGS,
            ...
            Root.COLUMN_AVAILABLE_BYTES
    };
```

如果您的文档提供程序连接到一组动态根目录（例如可能断开连接的 USB 设备或用户可以退出的帐号），则您可以使用 `ContentResolver.notifyChange()` 方法更新文档界面以同步这些更改，如下面的代码段所示。

```kotlin
    val rootsUri: Uri = DocumentsContract.buildRootsUri(BuildConfig.DOCUMENTS_AUTHORITY)
    context.contentResolver.notifyChange(rootsUri, null)
```

#### 在提供程序中列出文档

在实现 `queryChildDocuments()` 时，您必须使用 `DocumentsContract.Document` 中定义的列，返回指向指定目录中所有文件的 `Cursor`。

当用户在选择器界面中选择根目录时，系统会调用此方法。该方法检索由 `COLUMN_DOCUMENT_ID` 指定的文档 ID 的子级。每当用户选择文档提供程序中的子目录时，系统就会调用该方法。

此代码段使用所请求的列创建新光标，然后将父目录中每个直接子级的相关信息添加到该光标。子级可以是图片，也可以是另一个目录 - 可以是任何文件：

```java
    @Override
    public Cursor queryChildDocuments(String parentDocumentId, String[] projection,
                                  String sortOrder) throws FileNotFoundException {

        final MatrixCursor result = new
                MatrixCursor(resolveDocumentProjection(projection));
        final File parent = getFileForDocId(parentDocumentId);
        for (File file : parent.listFiles()) {
            // Adds the file's display name, MIME type, size, and so on.
            includeFile(result, null, file);
        }
        return result;
    }

    private static String[] resolveDocumentProjection(String[] projection) {
        return projection != null ? projection : DEFAULT_DOCUMENT_PROJECTION;
    }

    private static final String[] DEFAULT_DOCUMENT_PROJECTION = new String[]{
            Document.COLUMN_DOCUMENT_ID,
            Document.COLUMN_MIME_TYPE,
            Document.COLUMN_DISPLAY_NAME,
            Document.COLUMN_LAST_MODIFIED,
            Document.COLUMN_FLAGS,
            Document.COLUMN_SIZE
    };
    private void includeFile(MatrixCursor result, String docId, File file)
            throws FileNotFoundException {
        if (docId == null) {
            docId = getDocIdForFile(getContext(), file);
        } else {
            file = getFileForDocId(docId);
        }

        int flags = 0;

        if (file.isDirectory()) {
            if (file.isDirectory() && file.canWrite()) {
                flags |= Document.FLAG_DIR_SUPPORTS_CREATE;
            }
        } else if (file.canWrite()) {
            flags |= Document.FLAG_SUPPORTS_WRITE;
            flags |= Document.FLAG_SUPPORTS_DELETE;
        }

        final String displayName = file.getName();
        final String mimeType = getTypeForFile(file);

        if (mimeType.startsWith("image/")) {
            flags |= Document.FLAG_SUPPORTS_THUMBNAIL;
        }

        final MatrixCursor.RowBuilder row = result.newRow();
        if (row != null) {
            row.add(Document.COLUMN_DOCUMENT_ID, docId);
            row.add(Document.COLUMN_DISPLAY_NAME, displayName);
            row.add(Document.COLUMN_SIZE, file.length());
            row.add(Document.COLUMN_MIME_TYPE, mimeType);
            row.add(Document.COLUMN_LAST_MODIFIED, file.lastModified());
            row.add(Document.COLUMN_FLAGS, flags);
        }
    }
```

#### 获取文档信息

在实现 `queryDocument()` 时，您必须使用 `DocumentsContract.Document` 中定义的列，返回指向指定文件的 `Cursor`。

`queryDocument()` 方法返回的信息与 `queryChildDocuments()` 中传递的信息相同，但对于特定文件：

```java
    @Override
    public Cursor queryDocument(String documentId, String[] projection) throws
            FileNotFoundException {

        // Create a cursor with the requested projection, or the default projection.
        final MatrixCursor result = new
                MatrixCursor(resolveDocumentProjection(projection));
        includeFile(result, documentId, null);
        return result;
    }
```

您的文档提供程序还可以通过替换 `DocumentsProvider.openDocumentThumbnail()` 方法并将 `FLAG_SUPPORTS_THUMBNAIL` 标记添加到支持的文件来提供文档的缩略图。以下代码段提供了如何实现 `DocumentsProvider.openDocumentThumbnail()` 的示例。

```kotlin
    override fun openDocumentThumbnail(
            documentId: String?,
            sizeHint: Point?,
            signal: CancellationSignal?
    ): AssetFileDescriptor {
        val file = getThumbnailFileForDocId(documentId)
        val pfd = ParcelFileDescriptor.open(file, ParcelFileDescriptor.MODE_READ_ONLY)
        return AssetFileDescriptor(pfd, 0, AssetFileDescriptor.UNKNOWN_LENGTH)
    }
```

#### 打开文档 

您必须实现 `openDocument()` 以返回表示指定文件的 `ParcelFileDescriptor`。其他应用可以使用返回的 `ParcelFileDescriptor` 来流式传输数据。系统会在用户选择文件后调用此方法，并且客户端应用会通过调用 `openFileDescriptor()` 来请求访问该文件。

```java
    @Override
    public ParcelFileDescriptor openDocument(final String documentId,
                                             final String mode,
                                             CancellationSignal signal) throws
            FileNotFoundException {
        Log.v(TAG, "openDocument, mode: " + mode);
        // It's OK to do network operations in this method to download the document,
        // as long as you periodically check the CancellationSignal. If you have an
        // extremely large file to transfer from the network, a better solution may
        // be pipes or sockets (see ParcelFileDescriptor for helper methods).

        final File file = getFileForDocId(documentId);
        final int accessMode = ParcelFileDescriptor.parseMode(mode);

        final boolean isWrite = (mode.indexOf('w') != -1);
        if(isWrite) {
            // Attach a close listener if the document is opened in write mode.
            try {
                Handler handler = new Handler(getContext().getMainLooper());
                return ParcelFileDescriptor.open(file, accessMode, handler,
                            new ParcelFileDescriptor.OnCloseListener() {
                    @Override
                    public void onClose(IOException e) {

                        // Update the file with the cloud server. The client is done
                        // writing.
                        Log.i(TAG, "A file with id " +
                        documentId + " has been closed! Time to " +
                        "update the server.");
                    }

                });
            } catch (IOException e) {
                throw new FileNotFoundException("Failed to open document with id"
                + documentId + " and mode " + mode);
            }
        } else {
            return ParcelFileDescriptor.open(file, accessMode);
        }
    }
```

如果您的文档提供程序会流式传输文件或处理复杂的数据结构，则不妨考虑实现 `createReliablePipe()` 或 `createReliableSocketPair()` 方法。借助这些方法，您可以创建一对 `ParcelFileDescriptor` 对象，通过 `ParcelFileDescriptor.AutoCloseOutputStream` 或 `ParcelFileDescriptor.AutoCloseInputStream` 返回一个对象并发送另一个对象。

#### 支持最新文档和搜索

您可以通过替换 `queryRecentDocuments()` 方法并返回 `FLAG_SUPPORTS_RECENTS`，在文档提供程序的根目录下提供最近修改过的文档列表。以下代码段显示了如何实现 方法的示例。

```java
    @Override
    public Cursor queryRecentDocuments(String rootId, String[] projection)
            throws FileNotFoundException {

        // This example implementation walks a
        // local file structure to find the most recently
        // modified files.  Other implementations might
        // include making a network call to query a
        // server.

        // Create a cursor with the requested projection, or the default projection.
        final MatrixCursor result =
            new MatrixCursor(resolveDocumentProjection(projection));

        final File parent = getFileForDocId(rootId);

        // Create a queue to store the most recent documents,
        // which orders by last modified.
        PriorityQueue lastModifiedFiles =
            new PriorityQueue(5, new Comparator() {

            public int compare(File i, File j) {
                return Long.compare(i.lastModified(), j.lastModified());
            }
        });

        // Iterate through all files and directories
        // in the file structure under the root.  If
        // the file is more recent than the least
        // recently modified, add it to the queue,
        // limiting the number of results.
        final LinkedList pending = new LinkedList();

        // Start by adding the parent to the list of files to be processed
        pending.add(parent);

        // Do while we still have unexamined files
        while (!pending.isEmpty()) {
            // Take a file from the list of unprocessed files
            final File file = pending.removeFirst();
            if (file.isDirectory()) {
                // If it's a directory, add all its children to the unprocessed list
                Collections.addAll(pending, file.listFiles());
            } else {
                // If it's a file, add it to the ordered queue.
                lastModifiedFiles.add(file);
            }
        }

        // Add the most recent files to the cursor,
        // not exceeding the max number of results.
        for (int i = 0; i < Math.min(MAX_LAST_MODIFIED + 1, lastModifiedFiles.size()); i++) {
            final File file = lastModifiedFiles.remove();
            includeFile(result, null, file);
        }
        return result;
    }
```

您可以通过下载 [StorageProvider](https://github.com/android/storage-samples/tree/master/StorageProvider) 代码示例获取上述代码段的完整代码。

#### 支持文档管理功能

除了打开、创建和查看文件外，文档提供程序还允许客户端应用重命名、复制、移动和删除文件。要向文档提供程序添加文档管理功能，请在文档的 `COLUMN_FLAGS` 列中添加标记，以指明支持的功能。您还需要实现 `DocumentsProvider` 类的相应方法。

下表提供了文档提供程序提供特定功能所需实现的 `COLUMN_FLAGS` 标记和 `DocumentsProvider` 方法。

| 功能                                             | 标记                   | 方法               |
| :----------------------------------------------- | :--------------------- | :----------------- |
| 删除文件                                         | `FLAG_SUPPORTS_DELETE` | `deleteDocument()` |
| 重命名文件                                       | `FLAG_SUPPORTS_RENAME` | `renameDocument()` |
| 将文件复制到文档提供程序内的新父目录             | `FLAG_SUPPORTS_COPY`   | `copyDocument()`   |
|                                                  |                        |                    |
| 在文档提供程序内将文件从一个目录移动到另一个目录 | `FLAG_SUPPORTS_MOVE`   | `moveDocument()`   |
| 从某个文件的父目录中将其移除                     | `FLAG_SUPPORTS_REMOVE` | `removeDocument()` |

#### 支持虚拟文件和备用文件格式

略

### Security(安全性)

假设您的文档提供程序是受密码保护的云端存储服务，并且您希望先确保用户已登录，然后再开始共享文件。如果用户未登录，您的应用应该怎么做？解决之道是在您的 `queryRoots()` 实现中返回零个根目录。也就是空的根目录光标：

```java
    public Cursor queryRoots(String[] projection) throws FileNotFoundException {
    ...
        // If user is not logged in, return an empty root cursor.  This removes our
        // provider from the list entirely.
        if (!isUserLoggedIn()) {
            return result;
    }
    
```

或者调用`getCallingPackage()`检查正在访问的APP， 可验证不给除许可的app(包含DocumentUI, 因为我们可把Uri通过其它方式给其它客户端， 不一定通过它)使用；

下一步是调用 `getContentResolver().notifyChange()`；我们要使用它来创建此 URI。每当用户的登录状态发生变化时，以下代码段都会告知系统查询您的文档提供程序的根目录。如果用户未登录，则调用 `queryRoots()` 会返回空光标，如上所示。这有助于确保提供程序的文档仅在用户登录提供程序时可供查看。

```java
    private void onLoginButtonClick() {
        loginOrLogout();
        getContentResolver().notifyChange(DocumentsContract
                .buildRootsUri(AUTHORITY), null);
    }
```

