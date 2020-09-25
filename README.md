# swagger-pdf

## Swagger 

maven

```
	<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-boot-starter</artifactId>
			<version>3.0.0</version>
		</dependency>

```



Swagger 在3.0 以后移除了 /swagger-ui.html 更换成http://localhost:8080/swagger-ui/index.html

配置更改

* 在主类添加 @EnableOpenApi

  ```java
  package com.example.demo;
  
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  import springfox.documentation.oas.annotations.EnableOpenApi;
  import tk.mybatis.spring.annotation.MapperScan;
  
  @EnableOpenApi
  @MapperScan(basePackages =  "com.example.demo.mapper")
  @SpringBootApplication
  public class DemoApplication {
  
  	public static void main(String[] args) {
  		SpringApplication.run(DemoApplication.class, args);
  	}
  
  }
  
  ```

  

### 中文PDF导出



maven

```
	<!-- swagger  to pdf plugin -->
			<plugin>
				<groupId>io.github.swagger2markup</groupId>
				<artifactId>swagger2markup-maven-plugin</artifactId>
				<version>1.3.7</version>
				<configuration>
					<!--此处端口一定要是当前项目启动所用的端口-->
					<swaggerInput>http://localhost:8080/v2/api-docs</swaggerInput>
					<outputDir>target/docs/asciidoc/generated</outputDir>
					<config>
						<!-- 除了ASCIIDOC之外，还有MARKDOWN和CONFLUENCE_MARKUP可选 -->
						<swagger2markup.markupLanguage>ASCIIDOC</swagger2markup.markupLanguage>
					</config>
				</configuration>
			</plugin>

			<plugin>
				<groupId>org.asciidoctor</groupId>
				<artifactId>asciidoctor-maven-plugin</artifactId>
				<version>2.0.0-RC.1</version>
				<dependencies>
					<dependency>
						<groupId>org.asciidoctor</groupId>
						<artifactId>asciidoctorj-pdf</artifactId>
						<version>1.5.0-alpha.18</version>
					</dependency>
					<!-- Comment this section to use the default jruby artifact provided by the plugin -->
					<dependency>
						<groupId>org.jruby</groupId>
						<artifactId>jruby-complete</artifactId>
						<version>9.2.7.0</version>
					</dependency>
					<!-- Comment this section to use the default AsciidoctorJ artifact provided by the plugin -->
					<dependency>
						<groupId>org.asciidoctor</groupId>
						<artifactId>asciidoctorj</artifactId>
						<version>2.0.0</version>
					</dependency>
				</dependencies>
				<configuration>
					<sourceDirectory>target/docs/asciidoc/generated</sourceDirectory>
					<!-- Attributes common to all output formats -->
					<attributes>
						<sourcedir>target/docs/asciidoc/generated</sourcedir>
					</attributes>
				</configuration>
				<executions>
					<execution>
						<id>generate-pdf-doc</id>
						<phase>generate-resources</phase>
						<goals>
							<goal>process-asciidoc</goal>
						</goals>
						<configuration>
							<backend>pdf</backend>
							<!-- Since 1.5.0-alpha.9 PDF back-end can use 'rouge' as well as 'coderay'
                            for source highlighting -->
							<!-- Due to a known issue on windows, it is recommended to use 'coderay' until an new version of 'rouge' is released.
                            -->
							<sourceHighlighter>coderay</sourceHighlighter>
							<attributes>
								<icons>font</icons>
								<pagenums/>
								<toc/>
								<idprefix/>
								<idseparator>-</idseparator>
							</attributes>
						</configuration>
					</execution>
				</executions>
			</plugin>

```



```
mvn swagger2markup:convertSwagger2markup 
mvn generate-resources

```





### 问题思路

中文丢失或乱码，无非就是编码或者是字体文件导致的，后面查询了相关资料发现，asciidoctor-maven-plugin导出PDF所依赖的asciidoctorj-pdf工具包，里面自带的字体文件对中文支持不是很好，所以我们只要将他的字体文件替换掉就可解决这个问题。

### 问题解决

#### 【一】在maven仓库找到该工具包并找到字体文件所在位置

首先找到jar包：
我的路径在：D:\maven\repository\org\asciidoctor\asciidoctorj-pdf\1.5.0-alpha.10.1下面
![这里写图片描述](https://img-blog.csdn.net/20180727115314732?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NTM0NDgz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
找到jar之后，打开字体文件路径：asciidoctorj-pdf-1.5.0-alpha.10.1.jar\gems\asciidoctor-pdf-1.5.0.alpha.10\data\fonts
![这里写图片描述](https://img-blog.csdn.net/20180727115521479?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NTM0NDgz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 【二】替换字体文件

在fonts下面有以.ttf结尾的字体文件，在themes下面是指定主题的配置文件，使用哪个字体文件，就是在这里指定的，我们先把自己下载好的字体文件加进去：
![这里写图片描述](https://img-blog.csdn.net/20180727115912602?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NTM0NDgz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 【三】修改主题配置文件

再修改**asciidoctorj-pdf-1.5.0-alpha.10.1.jar\gems\asciidoctor-pdf-1.5.0.alpha.10\data\themes**下面的default-theme.yml配置文件：
![这里写图片描述](https://img-blog.csdn.net/20180727120322879?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NTM0NDgz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 【四】修改完，运行命令

修改完成之后，保存，再去执行生成命令：

```
mvn asciidoctor:process-asciidoc
mvn generate-resources12
```

### 修改成果

修改前：
![这里写图片描述](https://img-blog.csdn.net/20180727121019653?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NTM0NDgz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
修改后：
![这里写图片描述](https://img-blog.csdn.net/20180727120736776?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NTM0NDgz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
可以看到，新生成的PDF文件没有再出现乱码和字体丢失现象，问题完美解决！
