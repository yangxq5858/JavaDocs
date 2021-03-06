基于gradle的  spring boot 项目实战

# 1.HelloWorld项目

## 1.开发环境

下载gradle，至少4.8以上

http://services.gradle.org/distributions/

解压，设置path路径，变量名为`GRADLE_HOME`，变量值就是刚才所说的`D:\Dev\gradle-4.7`，如下

并把 %GRADLE_HOME%\bin 加入到path中





设置Gradle的下载的Jar包路径：

需要增加环境变量 GRADLE_USER_HOME。

![](images/搜狗截图20181029112454.png)





## 2.利用spring initializr，选择web，选择gradle project

## 3）修改build.gradle文件,设置中央仓库为阿里云的

```gradle
//builscript 中脚本优先执行
buildscript {
	ext {
		springBootVersion = '2.0.6.RELEASE'
	}

	repositories {
		//mavenCentral()
		maven {url "http://maven.aliyun.com/nexus/content/groups/public/"}
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.hx.springboot'
version = '0.0.1-SNAPSHOT'
//jdk 的版本
sourceCompatibility = 1.8

repositories {
	//mavenCentral()
	maven {url "http://maven.aliyun.com/nexus/content/groups/public/"}
}


dependencies {
    //compile('org.springframework.boot:spring-boot-starter-web')
	implementation('org.springframework.boot:spring-boot-starter-web')
	runtimeOnly('org.springframework.boot:spring-boot-devtools')
	testImplementation('org.springframework.boot:spring-boot-starter-test')
}

```

## 4）打开项目目录，在cmd命令模式下，运行 gradle build 命令 ，编译项目，就会下载依赖包

## 5）写一个controller

```java

@RestController
public class HelloController {


    @RequestMapping("/hello")
    public String hello(){
        return "hello world!";
    }
}
```

## 6）写测试，采用模拟mvc测试。

```java
package com.hxcoltd.demo.controller;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.hamcrest.Matchers.equalTo;
/**
 * @author yxqiang
 * @create 2018-10-26 21:20
 */

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class HelloControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    public void testHello() throws Exception {

        mockMvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
        .andExpect(content().string(equalTo("hello world!")));


        MvcResult rs = mockMvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON)).andReturn();
        System.out.println(rs.getResponse().getContentAsString());
    }

}

```

## 7）三种运行方式

1）java -jar xxx.jar （gradle build 编译出jar包）

2）gradle bootRun

3）IDE 里面右键项目运行



## 8）扩展学习

基于Spring boot的博客系统实战

http://coding.imooc.com/class/125.html



# 2.天气项目（集成了Redis，HttpClient，Jackson）

利用spring rest 客户端 RestTemplate 来发送http请求，返回一个ResponseEntity<T>对象

需要，先注入进来,写一个配置类

## 1）config 配置

```java
@Configuration
public class RestConfiguration {
	
	@Autowired
	private RestTemplateBuilder builder;

	@Bean
	public RestTemplate restTemplate() {
		return builder.build();
	}
	
}

```



## 2）controller

```java
package com.hxcoltd.weather.controller;

import com.hxcoltd.weather.service.WeatherDataService;
import com.hxcoltd.weather.vo.WeatherResponse;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author yxqiang
 * @create 2018-10-27 9:31
 */
@RestController
@RequestMapping("/weather")
public class WeatherController {
    @Autowired
    private WeatherDataService weatherDataService;

    @GetMapping("/cityId/{cityId}")
    public WeatherResponse getWeatherByCityId(@PathVariable("cityId") String cityId) {
        return weatherDataService.getDataByCityId(cityId);
    }

    @GetMapping("/cityName/{cityName}")
    public WeatherResponse getWeatherByCityName(@PathVariable("cityName") String cityName) {
        return weatherDataService.getDataByCityName(cityName);
    }
}


```

## 3）设置请求参数的默认值

```java
	@RequestMapping(value = {"/cityId/{cityId}", "/cityId"}, method = RequestMethod.GET)
	public ModelAndView getReportByCityId(@PathVariable(required = false)  String cityId, Model model) throws Exception {
		if (cityId == null) cityId = "101270101";
		model.addAttribute("title", "老杨的天气预报");
		model.addAttribute("cityId", cityId);
		model.addAttribute("cityList", cityDataService.listCity());
		model.addAttribute("report", weatherReportService.getDataByCityId(cityId));
		return new ModelAndView("weather/report", "reportModel", model);
	}
```


## 4）service
先定义接口，在定义class，面向接口编程


```java
package com.hxcoltd.weather.service;


import com.hxcoltd.weather.vo.WeatherResponse;

/**
 * Weather Data Service.
 * 
 * @since 1.0.0 2017年11月22日
 * @author <a href="https://waylau.com">Way Lau</a> 
 */
public interface WeatherDataService {
	/**
	 * 根据城市ID查询天气数据
	 * 
	 * @param cityId
	 * @return
	 */
	WeatherResponse getDataByCityId(String cityId);

	/**
	 * 根据城市名称查询天气数据
	 * 
	 * @param cityName
	 * @return
	 */
	WeatherResponse getDataByCityName(String cityName);
	
}

```

```java
package com.hxcoltd.weather.service;

import java.io.IOException;
import java.util.concurrent.TimeUnit;

import com.hxcoltd.weather.vo.WeatherResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

import com.fasterxml.jackson.databind.ObjectMapper;

/**
 * WeatherDataService 实现.
 * 
 * @since 1.0.0 2017年11月22日
 * @author <a href="https://waylau.com">Way Lau</a> 
 */
@Service
public class WeatherDataServiceImpl implements WeatherDataService {
    private static final Logger logger= LoggerFactory.getLogger(WeatherDataService.class);
	private static final String WEATHER_URI = "http://wthrcdn.etouch.cn/weather_mini?";
	private static final Long TIME_OUT = 10L;
	@Autowired
	private RestTemplate restTemplate;

	@Autowired
	private StringRedisTemplate stringRedisTemplate;
	
	@Override
	public WeatherResponse getDataByCityId(String cityId) {
		String uri = WEATHER_URI + "citykey=" + cityId;
		return this.doGetWeahter(uri);
	}

	@Override
	public WeatherResponse getDataByCityName(String cityName) {
		String uri = WEATHER_URI + "city=" + cityName;
		return this.doGetWeahter(uri);
	}

	/**
	 * 使用Redis，如果缓存有，使用缓存，没有远程访问，并加入到redis中
	 * @param uri
	 * @return
	 */
	private WeatherResponse doGetWeahter(String uri) {
		String key = uri;
		String strBody = null;
        WeatherResponse resp = null;
        ValueOperations<String, String> ops = stringRedisTemplate.opsForValue();

        if (stringRedisTemplate.hasKey(key)){
            logger.info("Redis has data");
            strBody = ops.get(key);

        }else{
            logger.info("Redis don't has data");
            // 缓存没有，再调用服务接口来获取

            //sprign rest 的一个客户端 restTemplate 用于将json序列化为一个ResponseEntity 的string对象
            ResponseEntity<String> respString = restTemplate.getForEntity(uri, String.class);


            if (respString.getStatusCodeValue() == 200) {
                strBody = respString.getBody();
            }

            // 数据写入缓存
            ops.set(key,strBody,TIME_OUT, TimeUnit.SECONDS);
        }

		try {

            //利用jackson来转换为对象
            ObjectMapper mapper = new ObjectMapper();

            //利用jackson来转换为对象
			resp = mapper.readValue(strBody, WeatherResponse.class);
		} catch (IOException e) {
		    logger.error("error！",e);
		}
		
		return resp;
	}

}

```

4) 编写vo类，用于实例化为第三方的数据

这里略过

# 3.集成Quartz定时服务

## 1)  depandecy

compile('org.springframework.boot:spring-boot-starter-quartz')

## 2）config 配置

```java
package com.waylau.spring.cloud.weather.config;

import org.quartz.JobBuilder;
import org.quartz.JobDetail;
import org.quartz.SimpleScheduleBuilder;
import org.quartz.Trigger;
import org.quartz.TriggerBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.waylau.spring.cloud.weather.job.WeatherDataSyncJob;

/**
 * Quartz Configuration.
 * 
 * @since 1.0.0 2017年11月23日
 * @author <a href="https://waylau.com">Way Lau</a> 
 */
@Configuration
public class QuartzConfiguration {

	private static final int TIME = 1800; // 更新频率

	// JobDetail
	@Bean
	public JobDetail weatherDataSyncJobDetail() {
		return JobBuilder.newJob(WeatherDataSyncJob.class).withIdentity("weatherDataSyncJob")
		.storeDurably().build();
	}
	
	// Trigger
	@Bean
	public Trigger weatherDataSyncTrigger() {
		
		SimpleScheduleBuilder schedBuilder = SimpleScheduleBuilder.simpleSchedule()
				.withIntervalInSeconds(TIME).repeatForever();
		
		return TriggerBuilder.newTrigger().forJob(weatherDataSyncJobDetail())
				.withIdentity("weatherDataSyncTrigger").withSchedule(schedBuilder).build();
	}
}

```

## 3) 编写job类

```java
package com.waylau.spring.cloud.weather.job;

import java.util.List;

import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.quartz.QuartzJobBean;

import com.waylau.spring.cloud.weather.service.CityDataService;
import com.waylau.spring.cloud.weather.service.WeatherDataService;
import com.waylau.spring.cloud.weather.vo.City;

/**
 * Weather Data Sync Job.
 * 
 * @since 1.0.0 2017年11月23日
 * @author <a href="https://waylau.com">Way Lau</a> 
 */
public class WeatherDataSyncJob extends QuartzJobBean {
	
	private final static Logger logger = LoggerFactory.getLogger(WeatherDataSyncJob.class);  
	
	@Autowired
	private CityDataService cityDataService;
	
	@Autowired
	private WeatherDataService weatherDataService;
	/* (non-Javadoc)
	 * @see org.springframework.scheduling.quartz.QuartzJobBean#executeInternal(org.quartz.JobExecutionContext)
	 */
	@Override
	protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
		logger.info("Weather Data Sync Job. Start！");
		// 获取城市ID列表
		List<City> cityList = null;
		
		try {
			cityList = cityDataService.listCity();
		} catch (Exception e) {
			logger.error("Exception!", e);
		}
		
		// 遍历城市ID获取天气
		for (City city : cityList) {
			String cityId = city.getCityId();
			logger.info("Weather Data Sync Job, cityId:" + cityId);
			
			weatherDataService.syncDateByCityId(cityId);
		}
		
		logger.info("Weather Data Sync Job. End！");
	}

}

```

# 4.存储城市列表（XML）

采用（javax.xml.bind.annotation）注解来解析xml数据

## 1）city bean

```java
package com.hxcoltd.weather.vo;
import javax.xml.bind.annotation.XmlAccessType;
import javax.xml.bind.annotation.XmlAccessorType;
import javax.xml.bind.annotation.XmlAttribute;
import javax.xml.bind.annotation.XmlRootElement;

/**
 * City.
 * 
 * @since 1.0.0 2017年11月23日
 * @author <a href="https://waylau.com">Way Lau</a> 
 */
@XmlRootElement(name = "d")
@XmlAccessorType(XmlAccessType.FIELD)
public class City {
	@XmlAttribute(name = "d1")
	private String cityId;
	
	@XmlAttribute(name = "d2")
	private String cityName;
	
	@XmlAttribute(name = "d3")
	private String cityCode;
	
	@XmlAttribute(name = "d4")
	private String province;

	public String getCityId() {
		return cityId;
	}

	public void setCityId(String cityId) {
		this.cityId = cityId;
	}

	public String getCityName() {
		return cityName;
	}

	public void setCityName(String cityName) {
		this.cityName = cityName;
	}

	public String getCityCode() {
		return cityCode;
	}

	public void setCityCode(String cityCode) {
		this.cityCode = cityCode;
	}

	public String getProvince() {
		return province;
	}

	public void setProvince(String province) {
		this.province = province;
	}
}

```

## 2）cityList类

```java
package com.hxcoltd.weather.vo;

import java.util.List;

import javax.xml.bind.annotation.XmlAccessType;
import javax.xml.bind.annotation.XmlAccessorType;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;

/**
 * City List.
 * 
 * @since 1.0.0 2017年11月23日
 * @author <a href="https://waylau.com">Way Lau</a> 
 */
@XmlRootElement(name = "c")
@XmlAccessorType(XmlAccessType.FIELD)
public class CityList {
	@XmlElement(name = "d")
	private List<City> cityList;

	public List<City> getCityList() {
		return cityList;
	}

	public void setCityList(List<City> cityList) {
		this.cityList = cityList;
	}
}

```

xml文件内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<c c1="0">
<d d1="101280101" d2="广州" d3="guangzhou" d4="广东"/>
<d d1="101280102" d2="番禺" d3="panyu" d4="广东"/>
<d d1="101280103" d2="从化" d3="conghua" d4="广东"/>
<d d1="101280104" d2="增城" d3="zengcheng" d4="广东"/>
<d d1="101280105" d2="花都" d3="huadu" d4="广东"/>
<d d1="101280201" d2="韶关" d3="shaoguan" d4="广东"/>
<d d1="101280202" d2="乳源" d3="ruyuan" d4="广东"/>
<d d1="101280203" d2="始兴" d3="shixing" d4="广东"/>
<d d1="101280204" d2="翁源" d3="wengyuan" d4="广东"/>
<d d1="101280205" d2="乐昌" d3="lechang" d4="广东"/>
<d d1="101280206" d2="仁化" d3="renhua" d4="广东"/>
<d d1="101280207" d2="南雄" d3="nanxiong" d4="广东"/>
<d d1="101280208" d2="新丰" d3="xinfeng" d4="广东"/>
<d d1="101280209" d2="曲江" d3="qujiang" d4="广东"/>
<d d1="101280210" d2="浈江" d3="chengjiang" d4="广东"/>
<d d1="101280211" d2="武江" d3="wujiang" d4="广东"/>
<d d1="101280301" d2="惠州" d3="huizhou" d4="广东"/>
<d d1="101280302" d2="博罗" d3="boluo" d4="广东"/>
<d d1="101280303" d2="惠阳" d3="huiyang" d4="广东"/>
<d d1="101280304" d2="惠东" d3="huidong" d4="广东"/>
<d d1="101280305" d2="龙门" d3="longmen" d4="广东"/>
<d d1="101280401" d2="梅州" d3="meizhou" d4="广东"/>
<d d1="101280402" d2="兴宁" d3="xingning" d4="广东"/>
<d d1="101280403" d2="蕉岭" d3="jiaoling" d4="广东"/>
<d d1="101280404" d2="大埔" d3="dabu" d4="广东"/>
<d d1="101280406" d2="丰顺" d3="fengshun" d4="广东"/>
<d d1="101280407" d2="平远" d3="pingyuan" d4="广东"/>
<d d1="101280408" d2="五华" d3="wuhua" d4="广东"/>
<d d1="101280409" d2="梅县" d3="meixian" d4="广东"/>
<d d1="101280501" d2="汕头" d3="shantou" d4="广东"/>
<d d1="101280502" d2="潮阳" d3="chaoyang" d4="广东"/>
<d d1="101280503" d2="澄海" d3="chenghai" d4="广东"/>
<d d1="101280504" d2="南澳" d3="nanao" d4="广东"/>
<d d1="101280601" d2="深圳" d3="shenzhen" d4="广东"/>
<d d1="101280701" d2="珠海" d3="zhuhai" d4="广东"/>
<d d1="101280702" d2="斗门" d3="doumen" d4="广东"/>
<d d1="101280703" d2="金湾" d3="jinwan" d4="广东"/>
<d d1="101280800" d2="佛山" d3="foshan" d4="广东"/>
<d d1="101280801" d2="顺德" d3="shunde" d4="广东"/>
<d d1="101280802" d2="三水" d3="sanshui" d4="广东"/>
<d d1="101280803" d2="南海" d3="nanhai" d4="广东"/>
<d d1="101280804" d2="高明" d3="gaoming" d4="广东"/>
<d d1="101280901" d2="肇庆" d3="zhaoqing" d4="广东"/>
<d d1="101280902" d2="广宁" d3="guangning" d4="广东"/>
<d d1="101280903" d2="四会" d3="sihui" d4="广东"/>
<d d1="101280905" d2="德庆" d3="deqing" d4="广东"/>
<d d1="101280906" d2="怀集" d3="huaiji" d4="广东"/>
<d d1="101280907" d2="封开" d3="fengkai" d4="广东"/>
<d d1="101280908" d2="高要" d3="gaoyao" d4="广东"/>
<d d1="101281001" d2="湛江" d3="zhanjiang" d4="广东"/>
<d d1="101281002" d2="吴川" d3="wuchuan" d4="广东"/>
<d d1="101281003" d2="雷州" d3="leizhou" d4="广东"/>
<d d1="101281004" d2="徐闻" d3="xuwen" d4="广东"/>
<d d1="101281005" d2="廉江" d3="lianjiang" d4="广东"/>
<d d1="101281006" d2="赤坎" d3="chikan" d4="广东"/>
<d d1="101281007" d2="遂溪" d3="suixi" d4="广东"/>
<d d1="101281008" d2="坡头" d3="potou" d4="广东"/>
<d d1="101281009" d2="霞山" d3="xiashan" d4="广东"/>
<d d1="101281010" d2="麻章" d3="mazhang" d4="广东"/>
<d d1="101281101" d2="江门" d3="jiangmen" d4="广东"/>
<d d1="101281103" d2="开平" d3="kaiping" d4="广东"/>
<d d1="101281104" d2="新会" d3="xinhui" d4="广东"/>
<d d1="101281105" d2="恩平" d3="enping" d4="广东"/>
<d d1="101281106" d2="台山" d3="taishan" d4="广东"/>
<d d1="101281107" d2="蓬江" d3="pengjiang" d4="广东"/>
<d d1="101281108" d2="鹤山" d3="heshan" d4="广东"/>
<d d1="101281109" d2="江海" d3="jianghai" d4="广东"/>
<d d1="101281201" d2="河源" d3="heyuan" d4="广东"/>
<d d1="101281202" d2="紫金" d3="zijin" d4="广东"/>
<d d1="101281203" d2="连平" d3="lianping" d4="广东"/>
<d d1="101281204" d2="和平" d3="heping" d4="广东"/>
<d d1="101281205" d2="龙川" d3="longchuan" d4="广东"/>
<d d1="101281206" d2="东源" d3="dongyuan" d4="广东"/>
<d d1="101281301" d2="清远" d3="qingyuan" d4="广东"/>
<d d1="101281302" d2="连南" d3="liannan" d4="广东"/>
<d d1="101281303" d2="连州" d3="lianzhou" d4="广东"/>
<d d1="101281304" d2="连山" d3="lianshan" d4="广东"/>
<d d1="101281305" d2="阳山" d3="yangshan" d4="广东"/>
<d d1="101281306" d2="佛冈" d3="fogang" d4="广东"/>
<d d1="101281307" d2="英德" d3="yingde" d4="广东"/>
<d d1="101281308" d2="清新" d3="qingxin" d4="广东"/>
<d d1="101281401" d2="云浮" d3="yunfu" d4="广东"/>
<d d1="101281402" d2="罗定" d3="luoding" d4="广东"/>
<d d1="101281403" d2="新兴" d3="xinxing" d4="广东"/>
<d d1="101281404" d2="郁南" d3="yunan" d4="广东"/>
<d d1="101281406" d2="云安" d3="yunan" d4="广东"/>
<d d1="101281501" d2="潮州" d3="chaozhou" d4="广东"/>
<d d1="101281502" d2="饶平" d3="raoping" d4="广东"/>
<d d1="101281503" d2="潮安" d3="chaoan" d4="广东"/>
<d d1="101281601" d2="东莞" d3="dongguan" d4="广东"/>
<d d1="101281701" d2="中山" d3="zhongshan" d4="广东"/>
<d d1="101281801" d2="阳江" d3="yangjiang" d4="广东"/>
<d d1="101281802" d2="阳春" d3="yangchun" d4="广东"/>
<d d1="101281803" d2="阳东" d3="yangdong" d4="广东"/>
<d d1="101281804" d2="阳西" d3="yangxi" d4="广东"/>
<d d1="101281901" d2="揭阳" d3="jieyang" d4="广东"/>
<d d1="101281902" d2="揭西" d3="jiexi" d4="广东"/>
<d d1="101281903" d2="普宁" d3="puning" d4="广东"/>
<d d1="101281904" d2="惠来" d3="huilai" d4="广东"/>
<d d1="101281905" d2="揭东" d3="jiedong" d4="广东"/>
<d d1="101282001" d2="茂名" d3="maoming" d4="广东"/>
<d d1="101282002" d2="高州" d3="gaozhou" d4="广东"/>
<d d1="101282003" d2="化州" d3="huazhou" d4="广东"/>
<d d1="101282004" d2="电白" d3="dianbai" d4="广东"/>
<d d1="101282005" d2="信宜" d3="xinyi" d4="广东"/>
<d d1="101282006" d2="茂港" d3="maogang" d4="广东"/>
<d d1="101282101" d2="汕尾" d3="shanwei" d4="广东"/>
<d d1="101282102" d2="海丰" d3="haifeng" d4="广东"/>
<d d1="101282103" d2="陆丰" d3="lufeng" d4="广东"/>
<d d1="101282104" d2="陆河" d3="luhe" d4="广东"/>
</c>
```

## 3）util 工具类

```java
package com.hxcoltd.weather.util;

import java.io.Reader;
import java.io.StringReader;

import javax.xml.bind.JAXBContext;
import javax.xml.bind.Unmarshaller;

/**
 * Xml Builder.
 * 
 * @since 1.0.0 2017年11月23日
 * @author <a href="https://waylau.com">Way Lau</a> 
 */
public class XmlBuilder {

	/**
	 * 将XML转为指定的POJO
	 * @param clazz
	 * @param xmlStr
	 * @return
	 * @throws Exception
	 */
	public static Object xmlStrToObject(Class<?> clazz, String xmlStr) throws Exception {
		Object xmlObject = null;
		Reader reader = null;
		JAXBContext context = JAXBContext.newInstance(clazz);
		
		// XML 转为对象的接口
		Unmarshaller unmarshaller = context.createUnmarshaller();
		
		reader = new StringReader(xmlStr);
		xmlObject = unmarshaller.unmarshal(reader);
		
		if (null != reader) {
			reader.close();
		}
		
		return xmlObject;
	}
}

```

## 4) 数据处理

```java
package com.hxcoltd.weather.service;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.List;

import com.hxcoltd.weather.util.XmlBuilder;
import com.hxcoltd.weather.vo.City;
import com.hxcoltd.weather.vo.CityList;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Service;


/**
 * City Data Service.
 * 
 * @since 1.0.0 2017年11月23日
 * @author <a href="https://waylau.com">Way Lau</a> 
 */
@Service
public class CityDataServiceImpl implements CityDataService {

	@Override
	public List<City> listCity() throws Exception {
		// 读取XML文件
		Resource resource = new ClassPathResource("citylist.xml");
		BufferedReader br = new BufferedReader(new InputStreamReader(resource.getInputStream(), "utf-8"));
		StringBuffer buffer = new StringBuffer();
		String line = "";
		
		while ((line = br.readLine()) !=null) {
			buffer.append(line);
		}
		
		br.close();
		
		// XML转为Java对象
		CityList cityList = (CityList) XmlBuilder.xmlStrToObject(CityList.class, buffer.toString());
		return cityList.getCityList();
	}

}

```



5) 用redis 桌面管理工具查看数据

redis-desktop-manager-0.8.8.384.exe

# 5.使用Thymeleaf模板引擎

Thymeleaf 模板引擎，是spring 提供的后端模板。

## 1）添加依赖

```gradle
// 添加 Spring Boot Thymeleaf Starter 的依赖
compile('org.springframework.boot:spring-boot-starter-thymeleaf')
```

## 2）controller

```java
package com.hxcoltd.weather.controller;

import com.hxcoltd.weather.service.CityDataService;
import com.hxcoltd.weather.service.WeatherReportService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.ModelAndView;

/**
 * Weather Report Controller.
 * 
 * @since 1.0.0 2017年11月24日
 * @author <a href="https://waylau.com">Way Lau</a> 
 */
@RestController
@RequestMapping("/report")
public class WeatherReportController {
	@Autowired
	private CityDataService cityDataService;
	
	@Autowired
	private WeatherReportService weatherReportService;
	
	@GetMapping("/cityId/{cityId}")
	public ModelAndView getReportByCityId(@PathVariable("cityId") String cityId, Model model) throws Exception {
		model.addAttribute("title", "老杨的天气预报");
		model.addAttribute("cityId", cityId);
		model.addAttribute("cityList", cityDataService.listCity());
		model.addAttribute("report", weatherReportService.getDataByCityId(cityId));
		return new ModelAndView("weather/report", "reportModel", model);
	}

}

```

## 3）配置热部署文件

在Resources文件夹下的application.properties文件中增加一个配置，关闭themyleaf的缓存

```properties
# 热部署的静态文件
spring.thymeleaf.cache=false
```

## 4）编写界面，运用Thymeleaf

引入bootstrap

http://getbootstrap.com/docs/4.1/getting-started/introduction/ 是官方示例

下面是它的html模板

```html
<!doctype html>
<html lang="en">
  <head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">

    <title>Hello, world!</title>
  </head>
  <body>
    <h1>Hello, world!</h1>

    <!-- Optional JavaScript -->
    <!-- jQuery first, then Popper.js, then Bootstrap JS -->
    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js" integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>
  </body>
</html>
```



编写report.html

注意，这里用到了Thymeleaf中的标签 th，用到了${} 来取model中的值

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport"
	content="width=device-width, initial-scale=1, shrink-to-fit=no">
<!-- Bootstrap CSS -->
<link rel="stylesheet"
	href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta.2/css/bootstrap.min.css"
	integrity="sha384-PsH8R72JQ3SOdhVi3uxftmaW6Vc51MKb0q5P2rRUpPvrszuE4W1povHYgTpBfshb"
	crossorigin="anonymous">

<title>老杨的天气预报</title>
</head>
<body>
	<div class="container">
		<div class="row">
			<h3 th:text="${reportModel.title}">yxqiang</h3>
			<select class="custom-select" id="selectCityId">
				<option th:each="city : ${reportModel.cityList}"
					th:value="${city.cityId}" th:text="${city.cityName}"
					th:selected="${city.cityId eq reportModel.cityId}"></option>
			</select>
		</div>
		<div class="row">
			<h1 class="text-success" th:text="${reportModel.report.city}">成都</h1>
		</div>
		<div class="row">
			<p>
				空气质量指数：<span th:text="${reportModel.report.aqi}"></span>
			</p>
		</div>
		<div class="row">
			<p>
				当前温度：<span th:text="${reportModel.report.wendu}"></span>
			</p>
		</div>
		<div class="row">
			<p>
				温馨提示：<span th:text="${reportModel.report.ganmao}"></span>
			</p>
		</div>
		<div class="row">
			<div class="card border-info" th:each="forecast : ${reportModel.report.forecast}">
				<div class="card-body text-info">
					<p class ="card-text" th:text="${forecast.date}">日期</p>
					<p class ="card-text" th:text="${forecast.type}">天气类型</p>
					<p class ="card-text" th:text="${forecast.high}">最高温度</p>
					<p class ="card-text" th:text="${forecast.low}">最低温度</p>
					<p class ="card-text" th:text="${forecast.fengxiang}">风向</p>
				</div>
			</div>
		</div>
	</div>



	<!-- jQuery first, then Popper.js, then Bootstrap JS -->
	<script src="https://code.jquery.com/jquery-3.2.1.slim.min.js"
		integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN"
		crossorigin="anonymous"></script>
	<script
		src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.3/umd/popper.min.js"
		integrity="sha384-vFJXuSJphROIrBnz7yo7oB41mKfc8JzQZiCq4NCceLEaO4IHwicKwpJf9c9IpFgh"
		crossorigin="anonymous"></script>
	<script
		src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta.2/js/bootstrap.min.js"
		integrity="sha384-alpBpkh1PFOepccYVYDB4do5UnbKysX5WZXm3XxPqe5iKTfUKjNkCk9SaVuEZflJ"
		crossorigin="anonymous"></script>
	<!-- Optional JavaScript -->
	<script type="text/javascript" th:src="@{/js/weather/report.js}"></script>
</body>
</html>
```



# 6.springCloud 

## 1）单体架构的优劣势

![](images/搜狗截图20181027203355.png)

## 2）微服务的设计原则

拆分足够微

轻量级通讯

领域驱动设计原则

单一职责原则

高内聚、低耦合

DevOps（技术栈相关人都有）

不限于技术栈

## 3）如何来设计微服务系统

服务拆分

服务注册（服务中心，可以管理各个小服务）

服务发现

服务消费

统一入口

配置管理（配置管理平台）

熔断机制

自动扩展（根据当前应用负荷，自动扩容）



## 4）SpringCloud可以解决那些问题

配置管理

服务注册

服务发现

断路器

智能路由

负载均衡

微代理

服务间调用

一次性令牌

控制总线

思维导图模板

全局锁

领导选举

分布式会话

集群状态

分布式消息

# 7.springCloud 章节

## 1）环境配置

![](images/搜狗截图20181028111731.png)

![](images/搜狗截图20181028111950.png)

![](images/搜狗截图20181028112113.png)

## 2）子项介绍

### Spring Cloud Config:

配置中心，利用git来集中管理程序的配置

### Spring Cloud Netflix

Spring Cloud Netflix（视频开源的框架），Spring 在它基础上，进行了封装。集成了众多的开源软件，包括Eureka，Hystrix，Zuul，Archaius等

### Spring Cloud Bus

消息总线，利用分布式消息将服务和服务实例连接在一起，用于在一个集群中传播状态的变化，比如配置更改的事件。可与Spring Cloud Config 联合实现 热部署。

### Spring Cloud for Cloud Foundry

（VMware 推出的，国内使用少）Pass云平台

### Spring Cloud Cluster

基于 Zookeeper、Redis、Hazelcast、Consul 实现的领导选举和平民状态模式的抽象和实现

### Spring Cloud Consul

基于 Hashicorp Consul实现的服务发现和配置管理

### Spring Cloud Security

在Zuul代理中为 OAth2 REST 客户端和认证头转发提供负载均衡

### Spring Cloud Sleuth

适用于 Spring Cloud 应用程序的分布式跟踪，与Zipkin、HTrace 和基于日志（例如ELK）的跟踪相兼容，可以对日志进行收集。

### Spring Cloud Data Flow

一种针对现代运行时可组合的微服务应用程序的云本地编排服务。易于使用的DSL、拖放式GUI和REST API 一起简化了基于微服务的数据管道的整体编排

### Spring Cloud Stream

一个轻量级的事件驱动的微服务框架来快速构建可以连接到外部系统的应用程序。使用Apache Kafaka或RabbitMQ在Spring Boot应用程序之间发送和接收消息的简单声明模型。

### Spring Cloud Stream App Starters

基于Spring Boot为外部系统提供Spring的集成。

### Spring Cloud Task App Starters

Spring Cloud Task App Starters 是Spring Boot 应用程序，可能是任何进程，包括Spring Batch作业，并可以在数据处理有限的时间终止

### Spring Cloud Connectors

便于PaaS应用在各种平台上连接到后端，像数据库和消息服务

### Spring Cloud Starters

基于Spring Boot 的项目，用以简化Spring Cloud的依赖管理。该项目已经终止，并且在Angel.SR2后的版本和其他项目合并

### Spring Cloud CLI

Spring Boot CLI插件用于在Groovy中快速创建Spring Cloud组件应用程序。

### Spring Cloud Contract

是一个总体项目，其中包含帮助用户成功实施消费者驱动契约（Consumer Driven Contracts)的解决方案。

### 扩展学习

http://cloud.spring.io/spring-cloud-static/Finchley.M2/

## 3）服务注册和发现

集成Eureka的Server和Client端

Eureka 特点：

服务注册和发现机制

和Spring Cloud无缝集成

高可用

开源

### 1.集成Eureka Server端

#### 1）修改gradle文件

spring-cloud-starter-netflix-eureka-server

```gradle
// buildscript 代码块中脚本优先执行
buildscript {

    // ext 用于定义动态属性
    ext {
        springBootVersion = '2.0.2.RELEASE'
    }

    // 使用了Maven的中央仓库及Spring自己的仓库（也可以指定其他仓库）
    repositories {
        // mavenCentral()
        maven { url "http://maven.aliyun.com/nexus/content/groups/public/" }
        maven { url "https://repo.spring.io/snapshot" }
        maven { url "https://repo.spring.io/milestone" }
    }

    // 依赖关系
    dependencies {

        // classpath 声明了在执行其余的脚本时，ClassLoader 可以使用这些依赖项
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

// 使用插件
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

// 指定了生成的编译文件的版本，默认是打成了 jar 包
group = 'com.hxcoltd.spring.cloud'
version = '1.0.0'

// 指定编译 .java 文件的 JDK 版本
sourceCompatibility = 1.8

// 使用了Maven的中央仓库及Spring自己的仓库（也可以指定其他仓库）
repositories {
    //mavenCentral()
    maven { url "http://maven.aliyun.com/nexus/content/groups/public/" }
    maven { url "https://repo.spring.io/snapshot" }
    maven { url "https://repo.spring.io/milestone" }
}

ext {
    springCloudVersion = '2.0.2.RELEASE'
}

// 依赖关系
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web:${springBootVersion}")
    // Eureka Server
    compile("org.springframework.cloud:spring-cloud-starter-netflix-eureka-server:${springCloudVersion}")

    // 该依赖用于测试阶段
    testCompile('org.springframework.boot:spring-boot-starter-test')
}

```

#### 2）application.properties

```properties
server.port: 8761

eureka.instance.hostname: localhost
eureka.client.registerWithEureka: false
eureka.client.fetchRegistry: false
eureka.client.serviceUrl.defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

eureka.server.enable-self-preservation=false
```

#### 3）修改主程序入口

```java
@SpringBootApplication
@EnableEurekaServer //设置为服务端
public class WeatherApplication {

    public static void main(String[] args) {
        SpringApplication.run(WeatherApplication.class, args);
    }
}
```



### 2.集成Eureka Client端

1）修改gradle文件

依赖：spring-cloud-starter-netflix-eureka-client

```gradle
//builscript 中脚本优先执行
buildscript {
    ext {
        springBootVersion = '2.0.2.RELEASE'
    }

    repositories {
        //mavenCentral()
        maven {url "http://maven.aliyun.com/nexus/content/groups/public/"}
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.hx.springboot'
version = '0.0.1-SNAPSHOT'
//jdk 的版本
sourceCompatibility = 1.8

repositories {
    //mavenCentral()
    maven {url "http://maven.aliyun.com/nexus/content/groups/public/"}
}


ext {
    springCloudVersion = '2.0.2.RELEASE'
}

dependencies {
    
	
	compile("org.springframework.boot:spring-boot-starter-web:${springBootVersion}")
    // Eureka client
    compile("org.springframework.cloud:spring-cloud-starter-netflix-eureka-client:${springCloudVersion}")

    //用于访问第三方的api接口
    //compile('org.apache.httpcomponents:httpclient:4.5.3')

	//集成redis
	//compile('org.springframework.boot:spring-boot-starter-data-redis')

    //集成quartz
    //compile('org.springframework.boot:spring-boot-starter-quartz')

    // 添加 Spring Boot Thymeleaf Starter 的依赖
    //compile('org.springframework.boot:spring-boot-starter-thymeleaf')

    //runtimeOnly('org.springframework.boot:spring-boot-devtools')
    //testImplementation('org.springframework.boot:spring-boot-starter-test')
	testCompile('org.springframework.boot:spring-boot-starter-test')
	
}

```



2）application.properties

```properties
spring.application.name: micro-weather-city-eureka
eureka.client.serviceUrl.defaultZone: http://localhost:8761/eureka/
```

3) 主程序入口修改

```java

@SpringBootApplication
@EnableDiscoveryClient
public class WeatherApplication {

    public static void main(String[] args) {
        SpringApplication.run(WeatherApplication.class, args);
    }
}
```

### 3.运行Eureka 各个客户端

进入各个文件夹根目录：

gradle build：生成jar文件

java -jar xxx\build\libs\xxxxx.jar --server.port=8081  :运行，在idea中默认运行的端口为8081



也可以在idea中 application.properties中增加一个配置 server.port=8088 ,来启动自定义端口

```properties
server.port=8083

spring.application.name: micro-weather-data-eureka
eureka.client.serviceUrl.defaultZone: http://localhost:8761/eureka/
```

## 4）服务消费

### 1.客户端发现模式

![](images/搜狗截图20181028145611.png)
**负载均衡在客户端**

### 2.服务端发现模式

![](images/搜狗截图20181028145814.png)

**负载均衡在服务端**

### 3.常见的微服务的消费者

#### 1）Apache HttpClient

![](images/搜狗截图20181028150248.png)

![](images/搜狗截图20181028150348.png)

![](images/搜狗截图20181028150436.png)

#### 2）Ribbon 

Spring  Cloud 提供的基于客户端负载均衡的工具

![](images/搜狗截图20181028150706.png)

![](images/搜狗截图20181028150745.png)

![](images/搜狗截图20181028150836.png)

![](images/搜狗截图20181028150917.png)

![](images/搜狗截图20181028151043.png)

#### 3）Feign 声明式客户端

1.依赖

```gradle
 //集成feign 客户端消费
	compile("org.springframework.cloud:spring-cloud-starter-openfeign:${springCloudVersion}")
```

2.修改启动类注解

```java
package com.hxcoltd.weather;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class WeatherApplication {

    public static void main(String[] args) {
        SpringApplication.run(WeatherApplication.class, args);
    }
}

```

3.定义一个接口，来获取数据

```java
package com.hxcoltd.weather.service;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

/**
 * @author yxqiang
 * @create 2018-10-28 15:22
 */

@FeignClient("micro-weather-city-eureka")
public interface CityClient {

    @GetMapping("/cities")
    public String CityList();
}

```

4.定义个Controller来使用

```java
package com.hxcoltd.weather.controller;

import com.hxcoltd.weather.service.CityClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author yxqiang
 * @create 2018-10-28 15:25
 */

@RestController
public class CityController {

    @Autowired
    private CityClient cityClient;

    @GetMapping("/cities")
    public String CityList(){
        return cityClient.CityList();

    }
}

```

5.修改application.properties

```properties
spring.application.name: micro-weather-eureka-client-feign
eureka.client.serviceUrl.defaultZone: http://localhost:8761/eureka/

feign.client.config.feignName.connect-timeout=5000
feign.client.config.feignName.read-timeout=5000

```



## 5）API网关

### 1.概述

集合多个api

统一API入口

![](images/搜狗截图20181028183103.png)

**不同的API适用于不同的应用。**

避免将内部信息泄露给外部

为微服务添加额外的安全层（权限也会在这一层来做）

支持混合通讯协议

降低构建微服务的复杂性

微服务模拟与虚拟化

**弊端：**

在架构上需要额外考虑更多的编排与管理

路由逻辑配置要进行统一的管理

**可能引发单点故障**

### 2.常见的实现网关的方式

#### **1) nginx 作为API网关**

![](images/搜狗截图20181028184105.png)



#### **2)Spring Cloud Zuul**

提供了认证、鉴权、限流、动态路由、监控、弹性、安全、负载均衡、协助单点压测、静态响应等边缘服务的框架。

#### 3）Kong提供微服务

在nginx基础上实现的

![](images/搜狗截图20181028184421.png)



### 3.如何集成Zuul

![](images/搜狗截图20181028184608.png)

1.添加依赖

//集成 api zuul 网关

```gradle
compile("org.springframework.cloud:spring-cloud-starter-netflix-zuul:${springCloudVersion}")
```



2.修改启动类

```java
// @EnableZuulProxy


@SpringBootApplication
@EnableDiscoveryClient
@EnableZuulProxy
public class WeatherApplication {

    public static void main(String[] args) {
        SpringApplication.run(WeatherApplication.class, args);
    }
}
```

3.修改application.properties

```properties
spring.application.name: micro-weather-eureka-client-zuul
eureka.client.serviceUrl.defaultZone: http://localhost:8761/eureka/
#设置代理路径
zuul.routes.hi.path: /hi/**
#设置转化地址，实现反向代理的功能
zuul.routes.hi.service-id=micro-weather-eureka-client
```

4.访问：

将hello这个jar 运行在8081端口上，

将hello-zuul jar 运行在8080上。

http://localhost:8080/hi/hello 相当于 转发到http://localhost:8081/hello 这个请求了。



5.改造以前的项目

新增一个网关项目，配置如下：

```properties
spring.application.name: micro-weather-eureka-client-zuul
eureka.client.serviceUrl.defaultZone: http://localhost:8761/eureka/
#设置代理路径
zuul.routes.city.path: /city/**
#设置转化地址，实现反向代理的功能
zuul.routes.city.service-id=micro-weather-city-eureka

zuul.routes.data.path: /data/**
zuul.routes.data.service-id=micro-weather-data-eureka
```

在网关中设置了city和data2个网关，对应各自的微服务

在天气预报项目中，原来用了2个feign指向了2个微服务的地址，统一改为上面定义的api网关微服务的名字：**micro-weather-eureka-client-zuul**

![](images/搜狗截图20181028193241.png)



## 6）微服务的集中化配置

### 1.概述

**配置分类（不同的类型）**：

​    源代码、文件、数据库连接、远程调用

​    开发环境、测试环境、预发布环境、生产环境

​    编译时、打包时、运行时

​    启动加载和动态加载

**配置中心要求：**    

面向可配置的编码

隔离性

一致性（相同的环境）

集中化配置

**Spring Cloud Config**

Config Server 端：基于git服务器来实现的，有版本的概念。

Config Client 端

### 2.Spring Cloud Config Server 端集成

#### 1)依赖

```gradle
compile("org.springframework.cloud:spring-cloud-config-server:${springCloudVersion}")
```

#### 2)修改主程序

```java
package com.hxcoltd.weather;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableDiscoveryClient
@EnableConfigServer //应用上 服务端配置
public class WeatherApplication {

    public static void main(String[] args) {
        SpringApplication.run(WeatherApplication.class, args);
    }
}

```

#### 3）修改application.properties配置文件

```properties
spring.application.name: micro-weather-config-server
spring.cloud.config.server.default-application-name=config-server
server.port= 8888

eureka.client.serviceUrl.defaultZone: http://localhost:8761/eureka/


#uri 后面的是 github的仓库地址，没有.git 后缀
spring.cloud.config.server.git.uri=https://github.com/yangxq5858/spring-cloud-microservices-development
#searchPaths: 表示仓库下的一级目录
spring.cloud.config.server.git.search-paths=config-repo

# 配置仓库的分支
spring.cloud.config.label=master
# 访问git仓库的用户名
#spring.cloud.config.server.git.username=xxxxoooo
# 访问git仓库的用户密码 如果Git仓库为公开仓库，可以不填写用户名和密码，如果是私有仓库需要填写
#spring.cloud.config.server.git.password=xxxxoooo
```



### 3.Spring Cloud Client 端集成
#### 1)依赖

```gradle
compile("org.springframework.cloud:spring-cloud-config-client:${springCloudVersion}")
```

#### 2)修改主程序

只需开启Eureka 客户端注解，即@EnableDiscoveryClient 即可。无需Server端还要配置一个

```java
package com.hxcoltd.weather;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableDiscoveryClient
public class WeatherApplication {

    public static void main(String[] args) {
        SpringApplication.run(WeatherApplication.class, args);
    }
}

```

#### 3）修改application.properties配置文件

```properties
spring.application.name: micro-weather-config-client

eureka.client.serviceUrl.defaultZone: http://localhost:8761/eureka/
#表示是xxx项目的dev的配置
spring.cloud.config.profile=dev
spring.cloud.config.uri=http://localhost:8888


```

#### 4）配置中心文件命名规则

![](images/搜狗截图20181028210408.png)

注意：比如，Config-Client端的项目名为：micro-weather-config-client

那么，开发环境的配置文件名就应该为micro-weather-config-client-dev.properties,否则，客户端无法解析value

#### 5）客户端获取值：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class WeatherApplicationTests {

    @Value("${auther}") //通过这个@value注解就能获取到github中的 xxx-dev.properties 下的auther的值了
    private String auther;

    @Test
    public void contextLoads() {
        System.out.println(auther);
        assertEquals("yxqiang", auther);
    }

}
```

## 7）熔断机制

Hystrix

### 1.原理

- 对该服务的调用执行熔断，对于后续请求，不再继续调用该目标服务，而是直接返回，从而可以快速释放资源
- 保护系统

### 2.实现的方式

**断路器**，当请求达到阈值时，直接返回，告诉调用者，不要再重试了，不调用目标服务，进行隔离。

**断路器模式：**

断路器状态：打开，关闭，半打开。

![](images/搜狗截图20181029195925.png)

![](images/搜狗截图20181029195925.png)



![](images/搜狗截图20181029200219.png)

![](images/搜狗截图20181029200605.png)

![](images/搜狗截图20181029201000.png)

![](images/搜狗截图20181029201238.png)

![](images/搜狗截图20181029201412.png)

### 3.Hystrix集成

#### 1）添加依赖 spring-cloud-starter-netflix-hystrix

```gradle
// 依赖关系
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web:${springBootVersion}")
    //集成服务发现与注册的客户端 Eureka client
    compile("org.springframework.cloud:spring-cloud-starter-netflix-eureka-client:${springCloudVersion}")
	
    //集成feign 客户端消费，微服务与微服务之间的调用
	compile("org.springframework.cloud:spring-cloud-starter-openfeign:${springCloudVersion}")
	
	//集成断路器
	compile("org.springframework.cloud:spring-cloud-starter-netflix-hystrix:${springCloudVersion}")

    // 该依赖用于测试阶段
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

#### 2) 修改主程序

添加 @EnableCircuitBreaker 注解

```java
package com.hxcoltd.weather;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
@EnableCircuitBreaker //表示启用断路器
public class WeatherApplication {

    public static void main(String[] args) {
        SpringApplication.run(WeatherApplication.class, args);
    }
}
```



#### 3）修改调用服务的地方

在方法上，标注 @HystrixCommand，设置失败的回调方法名及回调方法的实现

```java
@RestController
public class CityController {

    @Autowired
    private CityClient cityClient;

    @GetMapping("/cities")
    @HystrixCommand(fallbackMethod = "defaultCitys")
    public String CityList(){
        return cityClient.CityList();

    }
    
    //回调方法
    public String defaultCitys(){
        return "City Data Service is Down!";
    }
}
```



CityClient 服务不用修改

```java
@Service
@FeignClient("micro-weather-city-eureka")
public interface CityClient {

    @GetMapping("/cities")
    public String CityList();
}			
```

#### 4）测试

将 micro-weather-city-eureka 这个微服务，断开，就会出错，就会启动断路器。



#### 5）改造天气项目(在feign中启用断路器)

这次，不在Controller上改造了，而是在DataClient 这个Service接口上用forgin调用其他微服务的地方来处理。

**这里，就需要在application.properteis中增加一个配置**

```properties

#在feign中使用断路器，就需要启用
feign.hystrix.enabled=true
```



![](images/搜狗截图20181029210003.png)

```java
@Service
@FeignClient(name="micro-weather-eureka-client-zuul",fallback = DefaultDataClient.class)
public interface DataClient {

    /**
     * 获取城市列表
     * @return
     * @throws Exception
     */
    @GetMapping("city/cities")
    List<City> listCity() throws Exception;

    /**
     * 根据城市id查询天气信息
     * @param cityId
     * @return
     */
    @GetMapping("data/weather/cityId/{cityId}")
    public WeatherResponse getDataByCityId(@PathVariable("cityId") String cityId);

}
```

DefaultDataClient 类

**并且，要将该类，设置为一个bean，才能被Spring所托管**

```java
@Component
public class DefaultDataClient implements DataClient {
    @Override
    public List<City> listCity() throws Exception {
        //这里模拟一个数据返回
        List<City> cityList = null;
        cityList = new ArrayList<>();

        City city = new City();
        city.setCityId("101270101");
        city.setCityName("成都");
        cityList.add(city);

        city = new City();
        city.setCityId("101270102");
        city.setCityName("龙泉驿");
        cityList.add(city);

        return cityList;
    }

    @Override
    public WeatherResponse getDataByCityId(String cityId) {
        //这个对象太复杂，就直接返回null了。
        return null;
    }
}
```

改造前端UI：

![](images/搜狗截图20181029210936.png)



## 8）自动扩展

### 1.概述

![](images/搜狗截图20181029214419.png)



![](images/搜狗截图20181029214336.png)



![](images/搜狗截图20181029214704.png)

![](images/搜狗截图20181029214759.png)

![](images/搜狗截图20181029215351.png)

![](images/搜狗截图20181029215732.png)

![](images/搜狗截图20181029215844.png)

![](images/搜狗截图20181029215912.png)

![](images/搜狗截图20181029220009.png)

### 2.如何实现自动扩展

![](images/搜狗截图20181029220359.png)

![](images/搜狗截图20181029220531.png)



![](images/搜狗截图20181029220619.png)

![](images/搜狗截图20181029220916.png)

![](images/搜狗截图20181029221005.png)

![](images/搜狗截图20181029221038.png)

![](images/搜狗截图20181029221100.png)

![](images/搜狗截图20181029221119.png)



![](images/搜狗截图20181029221148.png)















































