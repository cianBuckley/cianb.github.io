---
layout: post
title: "The most simple Java Version Manager In the World"
subtitle: "Need to live with multiple java versions? Here is the easiest way"
tags: ["java"]
keywords: java, versioning, jm, rvm 
comments: true
---

As java gets moves on to frequent updates, we're finding ourselves having to maintain multiple versions at once. This here is just a super simple and basic bash function which does it for you. 

Throw this into your `~/.bashrc` or `~/.zshrc`. Sorry windows users!

```
function jm {
while getopts :v:l:s: opt; do
	case $opt in
		:)
			if [[ $OPTARG == "l" ]]; then
				echo $(/usr/libexec/java_home -V)
			fi
			if [[ $OPTARG == "v" ]]; then
				echo $(java -version)
			fi
			;;
		s) 
			export JAVA_HOME=`/usr/libexec/java_home -v $OPTARG`
			echo $(java -version)
			;;
	esac
done
}
```

3 commands:

* `jm -v` will tell you your current java version
* `jm -s 1.8` will set your version to 1.8 (local. Edit your .rc file for global)
* `jm -l` to list your currently installed versions 

Here's an example of the output of `jm -l`

```
➜  ~ jm -l
Matching Java Virtual Machines (3):
    14.0.1, x86_64:     "OpenJDK 14.0.1"        /Library/Java/JavaVirtualMachines/jdk-14.0.1.jdk/Contents/Home
    11.0.2, x86_64:     "OpenJDK 11.0.2"        /Library/Java/JavaVirtualMachines/openjdk-11.0.2.jdk/Contents/Home
    1.8.0_191, x86_64:  "Java SE 8"     /Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home
```

This wee script does not install new versions for you. I personally didn't think it was worth the effort since I don't need to do it so often. Plus, I tend to prefer to keep things light. If you're looking for a tool that does this for you, I'd recommend checking out either [Jenv](https://www.jenv.be/) or [Jabba](https://github.com/shyiko/jabba) 

