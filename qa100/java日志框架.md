在现在使用的java日志框架中，一般使用门面模式来进行日志框架的适配

* slf4j、Commons-logging是日志门面，只提供相关接口，可以支持log4j、logback、log4j2、JUL等日志框架