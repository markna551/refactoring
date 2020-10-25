# refactoring
From PA4-Chayapol https://github.com/Chayapol-c/PA4-Chayapol

# startDownload() method
In the `src\flashget\DownloadManipulator.java` class  
https://github.com/Chayapol-c/PA4-Chayapol/blob/master/src/flashget/DownloadManipulator.java  
consider this code:  
```java
    public void startDownload(String urlName, File out, long size, int threadNumber) {
        ...
        for (int i = 0; i < threadNumber; i++) {
            tasks[i] = new DownloadTask(urlName, out, size * i, size);
            if (i == threadNumber - 1) {
                tasks[i] = new DownloadTask(urlName, out, size * i, fileLength - (size * i));
            }
            //bind each threadBar with DownloadTask
            threadProgresses[i].progressProperty().bind(tasks[i].progressProperty());
            tasks[i].valueProperty().addListener(this::valueChange);
            executor.execute(tasks[i]);
        }
        //bind downloadBar with each DownloadTask
        downloadBar.progressProperty().bind(tasks[0].progressProperty().multiply(0.2).add(tasks[1].progressProperty().multiply(0.2)).add(
                tasks[2].progressProperty().multiply(0.2).add(tasks[3].progressProperty().multiply(0.2).add(tasks[4].progressProperty().multiply(0.2)))));
        executor.shutdown();
    }
```
* Problem:
  - bind() in downloadBar.progressProperty() it's too long, hard to read.
* Refactor: move downloadBar.progressProperty() in to loop .
```java
  public void startDownload(String urlName, File out, long size, int threadNumber) {
        ...
        for (int i = 0; i < threadNumber; i++) {
            tasks[i] = new DownloadTask(urlName, out, size * i, size);
            if (i == threadNumber - 1) {
                tasks[i] = new DownloadTask(urlName, out, size * i, fileLength - (size * i));
            }
            //bind each threadBar with DownloadTask
            threadProgresses[i].progressProperty().bind(tasks[i].progressProperty());
            tasks[i].valueProperty().addListener(this::valueChange);
            downloadBar.progressProperty().bind(tasks[i].progressProperty().multiply(0.2));
            executor.execute(tasks[i]);
        }
        executor.shutdown();
```

# call() method
In the `src\flashget\DownloadTask.java` class  
https://github.com/Chayapol-c/PA4-Chayapol/blob/master/src/flashget/DownloadTask.java  
consider this code:
```java

public Long call() {
    final int BUFFERSIZE = 1024;
    byte[] buffer = new byte[BUFFERSIZE];
    try {
            URLConnection connection = url.openConnection();
            ...
            BufferedInputStream in = new BufferedInputStream(connection.getInputStream());
            RandomAccessFile out = new RandomAccessFile(file, "rwd");
            out.seek(start);
            int read;
            long download = 0;
            while (download < size) {
                read = in.read(buffer, 0, 1024);
                out.write(buffer, 0, read);
                download += read;
                ...
            }
            in.close();
            out.close();
            return download; 
        } catch (IOException ioe) {
            Notification.showDialog("Cannot find file " + file);
        }
        return null;
    }
}
```
* Problem:
  - when you calls BufferedInputStream and RandomAccessFile you need to close() before try out.
* Refactor : Create try to check IOException in URLConnection first
```java
try {
      connection = url.openConnection();
    } catch (IOException ioe) {
      Notification.showDialog("Cannot find file " + file);
    }
```
* Refactor : move BufferedInputStream and RandomAccessFile to near try because if you do that you it will auto close() after out of try.
```java
      try (BufferedInputStream in = new BufferedInputStream(connection.getInputStream());
             RandomAccessFile out = new RandomAccessFile(file, "rwd");){
            ...
            out.seek(start);
            int read;
            long download = 0;
            while (download < size) {
                read = in.read(buffer, 0, 1024);
                out.write(buffer, 0, read);
                download += read;
                ...
            }
            return download; 
        } catch (IOException ioe) {
            Notification.showDialog("Cannot find file " + file);
        }
        return null;
    }
}
```
