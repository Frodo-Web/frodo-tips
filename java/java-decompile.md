# Java class decompilation
Lee Benfield's Java decompiler [CFR](http://www.benf.org/other/cfr) is straight forward and can even batch decompile jar container. 

1. Download the most recent version of CFR (crf 0.152 at the time of writing)
```
wget https://www.benf.org/other/cfr/cfr-0.152.jar
```
2. Run decompile with output into terminal
```
java -jar cfr-0.152.jarjavaclasstodecompiles.class
```
2a. Run this if you want to export it into a separate Java file
```
java -jar cfr-0.152.jar javaclasstodecompiles.class > javaclasstodecompiles.java
```
3. To decomplile a complete jar container
```
java -jar cfr-0.152.jar javacontainer.jar --outputdir ./javacontainer
```
Caveat: When using "--ouputdir" the directory path is not relative to the folder CFR runs in. So make sure you either use an absolute path or the "." for relative paths.

Alternativie decompiler: 
[javadecompilers.com](http://javadecompilers.com)
