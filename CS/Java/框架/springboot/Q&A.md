# Intellij IDEA运行报Command line is too long解法

+ 修改项目下 .idea\workspace.xml，找到标签 <component name="PropertiesComponent"> ， 在标签里加一行  <property name="dynamic.classpath" value="true" />

