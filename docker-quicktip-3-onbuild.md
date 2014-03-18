#Docker quicktip #3 �C ONBUILD
#Docker quicktip #3 - ONBUILD

Docker 0.8 came out today, with it a slew of fantastic enhancements. Today we��ll be looking at one of them: ONBUILD.

���죬Docker0.8�����ˣ��°������˺ܶ�ǿ������ԡ����ǽ������������һ����ONBUILD

ONBUILD is a new instruction for the Dockerfile. It is for use when creating a base image and you want to defer instructions to child images. For example:

ONBUILD��Dockerfile��������һ�����ONBUILDָ���������ڹ�������ʱ����ִ�У������������Ӿ�����ִ�С����磺

     1   # Dockerfile
     2   FROM busybox
     3   ONBUILD RUN echo "You won't see me until later"
 
 ` `
 
     1   docker build -t me/no_echo_here .
     2    
     3   Uploading context  2.56 kB
     4   Uploading context
     5   Step 0 : FROM busybox
     6   Pulling repository busybox
     7   769b9341d937: Download complete
     8   511136ea3c5a: Download complete
     9   bf747efa0e2f: Download complete
    10   48e5f45168b9: Download complete
    11    ---> 769b9341d937
    12   Step 1 : ONBUILD RUN echo "You won't see me until later"
    13    ---> Running in 6bf1e8f65f00
    14    ---> f864c417cc99
    15   Successfully built f864c417cc9

Here the onbuild instruction is read, not run, but stored for later use.

���Կ�������������ʱ������onbuildָ��������������û��ִ�У������Ժ�ִ�С�

Here is the later use:

�Ժ��������ִ�еģ�



    # Dockerfile
    FROM me/no_echo_here

` `

    docker build -t me/echo_here .
    Uploading context  2.56 kB
    Uploading context
    Step 0 : FROM cpuguy83/no_echo_here
     
    # Executing 1 build triggers
    Step onbuild-0 : RUN echo "You won't see me until later"
     ---> Running in ebfede7e39c8
    You won't see me until later
     ---> ca6f025712d4
     ---> ca6f025712d4
    Successfully built ca6f025712d4
    
The ONBUILD instruction only gets run when building the cpuguy83/echo_here image.

ONBUILDָ��ֻ�ڹ���cpuguy83/echo_here����ʱ��ִ�С�

ONBUILD gets run just after the FROM and before any other instructions in a child image.

ONBUILDָ����FROM�����ִ�У��������Ӿ�����κ����

You can also have multiple ONBUILD instructions.

ONBUILDָ�����ָ�������

Why would you want this? It turns out it��s pretty darn awesome, and powerful. I have a demo github repo setup for this: [Docker ONBUILD Demo](https://github.com/cpuguy83/docker-onbuild_demo)

ΪʲôҪ�����أ���ʵONBUILD�Ƿǳ�����һ��������ҷǳ�ǿ������Github�Ϸ���һ��[ONBUILD��Demo](https://github.com/cpuguy83/docker-onbuild_demo)

Before diving into this, I just want to say I��ve probably used ONBUILD a bit excessively here in order to get the point across for what ONBUILD does and what it can do, it��s up to you how to use it in your projects.

�ڿ�Demo֮ǰ������������Ϊ����ʾONBUILD�Ĺ��ܣ�����Demo�п����е�����ȵ�ʹ����ONBUILD�ˡ�������Լ�����Ŀ��Ҫ����ʹ�á�

    # Dockerfile
    FROM ubuntu:12.04
     
    RUN apt-get update -qq && apt-get install -y ca-certificates sudo curl git-core
    RUN rm /bin/sh && ln -s /bin/bash /bin/sh
     
    RUN locale-gen  en_US.UTF-8
    ENV LANG en_US.UTF-8
    ENV LANGUAGE en_US.UTF-8
    ENV LC_ALL en_US.UTF-8
     
    RUN curl -L https://get.rvm.io | bash -s stable
    ENV PATH /usr/local/rvm/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    RUN /bin/bash -l -c rvm requirements
    RUN source /usr/local/rvm/scripts/rvm && rvm install ruby
    RUN rvm all do gem install bundler
     
    ONBUILD ADD . /opt/rails_demo
    ONBUILD WORKDIR /opt/rails_demo
    ONBUILD RUN rvm all do bundle install
    ONBUILD CMD rvm all do bundle exec rails server

This Dockerfile is doing some initial setup of a base image. Installs Ruby and bundler. Pretty typical stuff. At the end are the ONBUILD instructions.

�����Dockerfile�У������һ����������ĳ�ʼ�����ã���װ��Ruby��bundler������һЩ���͵�����������ONBUILD�����

    ONBUILD ADD . /opt/rails_demo

Tells any child image to add everything in the current directory to /opt/rails_demo. Remember, this only gets run from a child image, that is when another image uses this one as a base (or FROM). And it just so happens if you look in the repo I have a skeleton rails app in rails_demo that has it��s own Dockerfile in it, we��ll take a look at this later.

����������Ӿ���ѵ�ǰĿ¼���������ݸ��Ƶ�/opt/rails_demoĿ¼�С����ס����Щ����ֻ�����Ӿ�����ִ�У���ν���Ӿ��񣬾���ͨ��FROMָ��ӵ�ǰ�������������ľ��񣩡�����㿴һ��rails-demo���git�⣬�ͻᷢ�֣�������һ��rails�����ܣ������Լ���Dockerfile����������ٿ����Dockerfile�ļ���

    ONBUILD WORKDIR /opt/rails_demo
    
Tells any child image to set the working directory to /opt/rails_demo, which is where we told ADD to put any project files
�����������Ӿ���ѹ���Ŀ¼���õ�/opt/rails_demo��Ҳ����֮ǰ������ADD������ļ���Ŀ��Ŀ¼��

    ONBUILD RUN rvm all do bundle install

Tells any child image to have bundler install all gem dependencies, because we are assuming a Rails app here.
�����������bundler���Ӿ����а�װ���б������������Ϊ����Ҫ���Ӿ����а�װһ��Rails����

    ONBUILD CMD rvm all do bundle exec rails server

Tells any child image to set the CMD to start the rails server

�����������Ӿ����е���CMD����������rails����

Ok, so let��s see this image build, go ahead and do this for yourself so you can see the output.

OK�������������������Ĺ�����������ִ����Щ�����������Ľ����

    git clone git@github.com:cpuguy83/docker-onbuild_demo.git
    cd docker-onbuild_demo
    docker build -t cpuguy83/onbuild_demo .
     
    Step 0 : FROM ubuntu:12.04
     ---> 9cd978db300e
    Step 1 : RUN apt-get update -qq && apt-get install -y ca-certificates sudo curl git-core
     ---> Running in b32a089b7d2d
    # output supressed
    ldconfig deferred processing now taking place
     ---> d3fdefaed447
    Step 2 : RUN rm /bin/sh && ln -s /bin/bash /bin/sh
     ---> Running in f218cafc54d7
     ---> 21a59f8613e1
    Step 3 : RUN locale-gen  en_US.UTF-8
     ---> Running in 0fcd7672ddd5
    Generating locales...
    done
    Generation complete.
     ---> aa1074531047
    Step 4 : ENV LANG en_US.UTF-8
     ---> Running in dcf936d57f38
     ---> b9326a787f78
    Step 5 : ENV LANGUAGE en_US.UTF-8
     ---> Running in 2133c36335f5
     ---> 3382c53f7f40
    Step 6 : ENV LC_ALL en_US.UTF-8
     ---> Running in 83f353aba4c8
     ---> f849fc6bd0cd
    Step 7 : RUN curl -L https://get.rvm.io | bash -s stable
     ---> Running in b53cc257d59c
    # output supressed
    ---> 482a9f7ac656
    Step 8 : ENV PATH /usr/local/rvm/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
     ---> Running in c4666b639c70
     ---> b5d5c3e25730
    Step 9 : RUN /bin/bash -l -c rvm requirements
     ---> Running in 91469dbc25a6
    # output supressed
    Step 10 : RUN source /usr/local/rvm/scripts/rvm && rvm install ruby
     ---> Running in cb4cdfcda68f
    # output supressed
    Step 11 : RUN rvm all do gem install bundler
     ---> Running in 9571104b3b65
    Successfully installed bundler-1.5.3
    Parsing documentation for bundler-1.5.3
    Installing ri documentation for bundler-1.5.3
    Done installing documentation for bundler after 3 seconds
    1 gem installed
     ---> e2ea33486d62
    Step 12 : ONBUILD ADD . /opt/rails_demo
     ---> Running in 5bef85f266a4
     ---> 4082e2a71c7e
    Step 13 : ONBUILD WORKDIR /opt/rails_demo
     ---> Running in be1a06c7f9ab
     ---> 23bec71dce21
    Step 14 : ONBUILD RUN rvm all do bundle install
     ---> Running in 991da8dc7f61
     ---> 1547bef18de8
    Step 15 : ONBUILD CMD rvm all do bundle exec rails server
     ---> Running in c49139e13a0c
     ---> 23c388fb84c1
    Successfully built 23c388fb84c1
    
Now let��s take a look at that Dockerfile in the rails_demo project:
��������������rails_demo��Ŀ��Dockerfile��

    # Dockerfile
    FROM cpuguy83/onbuild_demo

WAT?? This Dockerfile is a grand total of one line. It��s only one line because we setup everything in the base image. The only pre-req is that the Dockerfile is built from within the Rails project tree. When we build this image, the ONBUILD commands from cpuguy83/onbuild_demo will be inserted just after the FROM instruction here.

ʲô��Dockerfile�ܹ�ֻ����ôţ��������һ�У�ȷʵ��ˣ�ֻ��һ�У���Ϊ���ǰ��������ö����ڸ��������ˡ��Ӿ���ֻ��Ҫ��֤Dockerfile�Ǵ�Rails��Ŀ�������������ļ��ɡ������ǹ����������ʱ��cpuguy83/onbuild_demo�����е�ONBUILD����ͻ���뵽FROMָ����档

����ע��Rails��Ŀ��Ӧ����ָ���ߵ�rails-demo�ľ������Ӿ������γɵ�һ�����νṹ��ֻҪDockerfile��ָ���ĸ�������������νṹ�е��κ�һ���ڵ㡣����rails-demo��ָ����ONBUILDָ������������Ӿ����ִ��ONBUILD��ָ�������

Remember, this aggressive use of ONBUILD may not be optimal for your project and is for demo purposes�� not to say it��s not ok 

��ע�⣬���ֹ���ʹ��ONBUILD�ķ�ʽ��һ���ʺ������Ŀ����������Ҳ��Ϊ��Demo����Ȼ��Ҳ������˵���ַ�ʽһ����OK���ٺ�


So let��s run this:
���ˣ�������ִ�й�������ɣ�

    cd rails_demo
    docker build -t cpuguy83/rails_demo .
     
    Step onbuild-0 : ADD . /opt/rails_demo
     ---> 11c1369a8926
    Step onbuild-1 : WORKDIR /opt/rails_demo
     ---> Running in 82def1878360
     ---> 39f8280cdca6
    Step onbuild-2 : RUN rvm all do bundle install
     ---> Running in 514d5fc643f1
    # output supressed
    Step onbuild-3 : CMD rvm all do bundle exec rails server
     ---> Running in df4a2646e4d9
     ---> b78c1813bd44
     ---> b78c1813bd44
    Successfully built b78c1813bd44
    
Then we can run the rails_demo image and have the rails server fire right up

������ɺ����ǾͿ�������rails_demo�����ˣ����е�rails������Ҳ���������ˡ�
    
    docker run -i -t cpuguy83/rails_demo
     
    => Booting WEBrick
    => Rails 3.2.14 application starting in development on http://0.0.0.0:3000
    => Call with -d to detach
    => Ctrl-C to shutdown server
    [2014-02-06 11:53:20] INFO  WEBrick 1.3.1
    [2014-02-06 11:53:20] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
    [2014-02-06 11:53:20] INFO  WEBrick::HTTPServer#start: pid=193 port=3000
    
TLDR; ONBUILD�� awesome. Use it to defer build instructions to images built from a base image. Use it to more easily build images from a common base but differ in some way, such as different git branches, or different projects entirely.

�˴�ʡ��������ǧ��...ONBUILD���ţ��ܰ�����������Ѿ�����ָ���������ӳٵ��Ӿ�����ȥִ�С���ONBUILD���Դ�һ��ͨ�õĸ������и������������ĳЩ����������ͬ���Ӿ��������Ӿ�������в�ͬ��֧������������Ŀ����ȫ�µġ�

With great power comes great responsibility.

����Խ������Խ�������ONBUILD��

