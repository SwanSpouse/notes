# Dockerfile

Dockerfile是一个文本格式的配置文件.Dockerfile文件分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

## FROM

指定所创建镜像的基础镜像，如果本地不存在，则默认会去Docker Hub下载指定镜像。

格式为FROM&lt;image&gt;或者FROM&lt;image&gt;:&lt;tag&gt;

## MAINTAINER

指定维护者信息，格式为MAINTAINER&lt;name&gt;

## RUN

运行指定命令

格式为RUN&lt;command&gt;或者RUN \[""executable", "param1","param2"\]

前者默认将在shell终端中运行命令，即/bin/sh -c； 后者使用exec执行，不会启动shell环境。

每条RUN指令将在当前镜像的基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用\来换行。

## CMD

CMD命令用来指定启动容器时默认执行的命令。它支持3中格式

* CMD \[""executable", "param1","param2"\] 使用exec执行，是推荐使用的方式；
* CMD command param1 param2 在/bin/sh中执行，提供给需要交互的应用；
* CMD \["param1", "param2"\]提供给ENTRYPOINT的默认参数。

每个dockerfile只能有一条CMD命令。如果指定了多条命令，只有最后一条会被执行。如果用户启动容器时，手动指定了运行的命令，则会覆盖掉CMD指定的命令。

## LABEL

LABEL指令用来指定生成镜像的元数据标签信息。

格式为LABEL &lt;key&gt;=&lt;value&gt; &lt;key&gt;=&lt;value&gt; ...

## EXPOSE

声明镜像内服务所监听的端口。格式为EXPOSE &lt;port&gt; \[&lt;port&gt;...\]

该指令只是起到声明作用，并不会自动完成端口映射。

## ENV

指定环境变量，在镜像生成过程中会被后续的RUN指令所使用，在镜像启动的容器中也会存在。

格式为ENV&lt;key&gt;&lt;value&gt;或ENV&lt;key&gt;=&lt;value&gt;

## ADD

该命令将复制指定的&lt;src&gt;路径下的内容到容器中的&lt;dest&gt;路径下

格式为ADD&lt;src&gt;&lt;dest&gt;

## COPY

格式为COPY&lt;src&gt;&lt;dest&gt;

复制本地主机的&lt;src&gt;下的内容到镜像中的&lt;dest&gt;下，目标路径不存在时，会自动创建。

## ENTRYPOINT

指定镜像的默认入口命令，该入口命令会在启动容器时作为根命令执行，所有传入值作为该命令的参数。

ENTRYPOINT \[""executable", "param1","param2"\] 使用exec执行

ENTRYPOINT command param1 param2 shell中执行。

## VOLUME

创建一个数据卷挂载点

格式为VOLUME \["/data"\]

可以从本地主机或者其他容器挂在数据卷，一般用来存放数据库和所需要保存的数据等。

## USER

指定容器运行时的用户名或UID，后续的RUN等指令也会使用指定的用户身份。

格式为USER daemon

当服务不需要管理员权限时，可以通过该命令指定运行用户，并且可以在之前创建所需要的用户。

## WORKDIR

为后续的RUN、CMD和ENTRYPOINT指定配置工作目录。

格式为 WORKDIR /path/to/workdir

## ARG

指定一些镜像内使用的参数，例如版本号等，这些参数在执行docker build命令时才以--build-arg&lt;varname&gt;=&lt;value&gt;格式传入。

格式为ARG&lt;name&gt;=\[=&lt;default value&gt;\]

## ONBUILD

配置当所创建的镜像作为其他镜像的基础镜像时，所执行的创建操作指令。

格式为ONBUILD \[INSTRUCTION\]

## STOPSIGNAL

指定所创建镜像启动的容器接受退出的信号值。

## HEALTHCHECK

配置所启动容器如何进行健康检查。

HEALTHCHECK \[OPTIONS\] CMD command：根据所执行的命令返回值是否为0来判断。

HEALTHCHECK NONE: 禁止基础镜像中的健康检查。

## SEHLL

指定其他命令使用shell时的默认shell类型。

格式SEHLL \["executable","parameters"\]

默认值为\["/bin/sh", "-c"\]

