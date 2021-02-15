JDKPATH = /Users/venkataramanabalajirajendran/Library/Java/JavaVirtualMachines/openjdk-15.0.2/Contents/Home
LIBPATH = /Users/venkataramanabalajirajendran/Documents/Venkat/MS/DBMSI/minjava/javaminibase/src
CLASSPATH = $(JDKPATH)/lib/classes.zip:.:..:$(LIBPATH)
BINPATH = $(JDKPATH)/bin
JAVAC = $(JDKPATH)/bin/javac -classpath $(CLASSPATH)
JAVA  = $(JDKPATH)/bin/java  -classpath $(CLASSPATH)

PROGS = together

all: $(PROGS)

together:*.java
	$(JAVAC) *.java

clean:
	\rm -f *.class *~ \#* core 
