---
layout: post
title: Metrics for your Spring REST API
categories: [java, spring]
tags: [java, spring, REST]
description: Metrics for your Spring REST API
---

*본 포스트의 원문은 [http://www.baeldung.com](http://www.baeldung.com/spring-rest-api-metrics)에서 확인할 수 있으며 원 저작자의 번역 포스팅 허락을 받았음을 알려드립니다.*

# 1. Overview

이번 예제에서는 Spring Rest API에 기본적인 Metrics(측정) 설정을 살펴보겠습니다.

metric 기능을 처음에는 간단한 Servlet Filter들을 활용해 보고, 다음으로 Spring Boot Actuator를 사용하겠습니다.

# 2. The web.xml

앱의 web.xml에 MetricFilter를 등록하면 됩니다. 

```
<filter>
    <filter-name>metricFilter</filter-name>
    <filter-class>org.baeldung.web.metric.MetricFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>metricFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```


"/*" 패턴 으로 들어오는 모든 request에 대해 필터를 매핑되도록 설정 하였습니다. 


# 3. The Servlet Filter

이제 custom 필터를 만들 차례입니다. 


{% highlight java %}
public class MetricFilter implements Filter {
 
    private MetricService metricService;
 
    @Override
    public void init(FilterConfig config) throws ServletException {
        metricService = (MetricService) WebApplicationContextUtils
         .getRequiredWebApplicationContext(config.getServletContext())
         .getBean("metricService");
    }
 
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
      throws java.io.IOException, ServletException {
        HttpServletRequest httpRequest = ((HttpServletRequest) request);
        String req = httpRequest.getMethod() + " " + httpRequest.getRequestURI();
 
        chain.doFilter(request, response);
 
        int status = ((HttpServletResponse) response).getStatus();
        metricService.increaseCount(req, status);
    }
}
{% endhighlight %}



필터는 일반적인 빈이 아니기 때문에 metricService에 inject 할 수 없지만 대신 ServletContext를 통해 직접 검색할 수 있습니다. 


그리고 doFilter API를 호출하여 filter chain을 실행합니다. 


# 4. Metric – Status Code Counts

다음으로 간단한 MetricService를 살펴보겠습니다. 


{% highlight java %}
@Service
public class MetricService {
 
    private Map<Integer, Integer> statusMetric;
 
    public void increaseCount(String request, int status) {
        Integer statusCount = statusMetric.get(status);
        if (statusCount == null) {
            statusMetric.put(status, 1);
        } else {
            statusMetric.put(status, statusCount + 1);
        }
    }
 
    public Map getStatusMetric() {
        return statusMetric;
    }
}
{% endhighlight %}




memory Map을 사용하여 각 Http status code를 카운팅하여 저장합니다. 

이제 기본적인 metric을 출력하기 위해 Controller 메소드에 매핑하도록 하겠습니다. 



{% highlight java %}
@RequestMapping(value = "/status-metric", method = RequestMethod.GET)
@ResponseBody
public Map getStatusMetric() {
    return metricService.getStatusMetric();
}
{% endhighlight %}



그러면 아래와 같은 간단한 응답을 확인할 수 있습니다. 


```
{  
    "404":1,
    "200":6,
    "409":1
}
```


# 5. Metric – Status Codes by Request


Next – let’s record metrics for Counts by Request:
다음은 Request 카운트 metric을 기록하도록 하겠습니다. 



{% highlight java %}
@Service
public class MetricService {
 
    private Map<String, HashMap<Integer, Integer>> metricMap;
 
    public void increaseCount(String request, int status) {
        HashMap<Integer, Integer> statusMap = metricMap.get(request);
        if (statusMap == null) {
            statusMap = new HashMap<Integer, Integer>();
        }
 
        Integer count = statusMap.get(status);
        if (count == null) {
            count = 1;
        } else {
            count++;
        }
        statusMap.put(status, count);
        metricMap.put(request, statusMap);
    }
 
    public Map getFullMetric() {
        return metricMap;
    }
}
{% endhighlight %}



API호출을 통해 metric을 출력하면 됩니다 :



{% highlight java %}
@RequestMapping(value = "/metric", method = RequestMethod.GET)
@ResponseBody
public Map getMetric() {
    return metricService.getFullMetric();
}
{% endhighlight %}



metric들을 아래와 같은 형태의 확인할 수 있습니다:

```
{
    "GET /users":
    {
        "200":6,
        "409":1
    },
    "GET /users/1":
    {
        "404":1
    }
}
```


According to the above example the API had the following activity:
위 예제를 통


"GET /users"로  "7" 번 request가 있었고
"6" 번은 "200" status code 응답으로 나갔고 한 번의 "409" 응답이 있었습니다. 


# 6. Metric – Time Series Data

application의 전반적인 카운트 수치들은 유용하지만 만약 오랜 기간동안 돌아가는 시스템에서는 *이런 metrics들(이전 예제에서 측정한 metrics)이 실질적으로 어떤 의미를 가지는지 알기 어렵습니다*

쉽게 이해하고 해석할 수 있는 데이터를 위해 시간 context가 필요합니다. 


간단한 time-based metric을 만들어 보겠습니다. *분당* status code 카운트 수를 기록하도록 하겠습니다 :



{% highlight java %}
@Service
public class MetricService{
 
    private Map<String, HashMap<Integer, Integer>> timeMap;
    private static SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm");
 
    public void increaseCount(String request, int status) {
        String time = dateFormat.format(new Date());
        HashMap<Integer, Integer> statusMap = timeMap.get(time);
        if (statusMap == null) {
            statusMap = new HashMap<Integer, Integer>();
        }
 
        Integer count = statusMap.get(status);
        if (count == null) {
            count = 1;
        } else {
            count++;
        }
        statusMap.put(status, count);
        timeMap.put(time, statusMap);
    }
}
{% endhighlight %}



그리고 getGraphData() 는 아래와 같습니다:




{% highlight java %}
public Object[][] getGraphData() {
    int colCount = statusMetric.keySet().size() + 1;
    Set<Integer> allStatus = statusMetric.keySet();
    int rowCount = timeMap.keySet().size() + 1;
    Object[][] result = new Object[rowCount][colCount];
    result[0][0] = "Time";
 
    int j = 1;
    for (int status : allStatus) {
        result[0][j] = status;
        j++;
    }
    int i = 1;
    Map<Integer, Integer> tempMap;
    for (Entry<String, HashMap<Integer, Integer>> entry : timeMap.entrySet()) {
        result[i][0] = entry.getKey();
        tempMap = entry.getValue();
        for (j = 1; j < colCount; j++) {
            result[i][j] = tempMap.get(result[0][j]);
            if (result[i][j] == null) {
                result[i][j] = 0;
            }
        }
        i++;
    }
 
    return result;
}
{% endhighlight %}



API에 매핑해 보도록 하겠습니다.



{% highlight java %}
@RequestMapping(value = "/metric-graph-data", method = RequestMethod.GET)
@ResponseBody
public Object[][] getMetricData() {
    return metricService.getGraphData();
}
{% endhighlight %}



마지막으로 Google Charts를 통해 렌더링 합니다. 



{% highlight html %}
<html>
<head>
<title>Metric Graph</title>
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
<script type="text/javascript" src="https://www.google.com/jsapi"></script>
<script type="text/javascript">
google.load("visualization", "1", {packages : [ "corechart" ]});
 
function drawChart() {
$.get("/metric-graph-data",function(mydata) {
    var data = google.visualization.arrayToDataTable(mydata);
    var options = {title : 'Website Metric',
                   hAxis : {title : 'Time',titleTextStyle : {color : '#333'}},
                   vAxis : {minValue : 0}};
 
    var chart = new google.visualization.AreaChart(document.getElementById('chart_div'));
    chart.draw(data, options);
 
});
 
}
</script>
</head>
<body onload="drawChart()">
    <div id="chart_div" style="width: 900px; height: 500px;"></div>
</body>
</html>
{% endhighlight %}



# 7. Using Spring Boot Actuator


다음 섹션들은 기존 metrics들을 표현하기 위해 Spring Boot의 Actuator 기능을 사용하겠습니다.

우선 pom.xml에 actuator dependency를 추가합니다:


```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```


# 8. The MetricFilter

다음은 MetricFilter를 실제 Spring bean으로 전환하겠습니다. 



{% highlight java %}
@Component
public class MetricFilter implements Filter {
 
    @Autowired
    private MetricService metricService;
 
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
      throws java.io.IOException, ServletException {
        chain.doFilter(request, response);
 
        int status = ((HttpServletResponse) response).getStatus();
        metricService.increaseCount(status);
    }
}
{% endhighlight %}



이전과 같이 dependicies를 직접 작성하는 것을 단순화 시켰습니다. 


# 9. Using CounterService

이제 각 Status Code 횟수를 카운팅하기 위해 CounterService를 사용하겠습니다.



{% highlight java %}
@Service
public class MetricService {
 
    @Autowired
    private CounterService counter;
 
    private List<String> statusList;
 
    public void increaseCount(int status) {
        counter.increment("status." + status);
        if (!statusList.contains("counter.status." + status)) {
            statusList.add("counter.status." + status);
        }
    }
}
{% endhighlight %}



# 10. Export Metrics using MetricRepository

이제 MetricRepository를 사용하여 metrics들을 export 하겠습니다:



{% highlight java %}
@Service
public class MetricService {
 
    @Autowired
    private MetricRepository repo;
 
    private List<ArrayList<Integer>> statusMetric;
    private List<String> statusList;
     
    @Scheduled(fixedDelay = 60000)
    private void exportMetrics() {
        Metric<?> metric;
        ArrayList<Integer> statusCount = new ArrayList<Integer>();
        for (String status : statusList) {
            metric = repo.findOne(status);
            if (metric != null) {
                statusCount.add(metric.getValue().intValue());
                repo.reset(status);
            } else {
                statusCount.add(0);
            }
        }
        statusMetric.add(statusCount);
    }
}
{% endhighlight %}



분당 status code들을 카운트하여 저장 합니다. 

# 11. Draw Graph using Metrics

마지막으로 2차원 배열로 metric들을 출력합니다. - 이를 통해 그래프를 그릴 수 있습니다. 



{% highlight java %}
public Object[][] getGraphData() {
    Date current = new Date();
    int colCount = statusList.size() + 1;
    int rowCount = statusMetric.size() + 1;
    Object[][] result = new Object[rowCount][colCount];
    result[0][0] = "Time";
 
    int j = 1;
    for (String status : statusList) {
        result[0][j] = status;
        j++;
    }
 
    ArrayList<Integer> temp;
    for (int i = 1; i < rowCount; i++) {
        temp = statusMetric.get(i - 1);
        result[i][0] = dateFormat.format
          (new Date(current.getTime() - (60000 * (rowCount - i))));
        for (j = 1; j <= temp.size(); j++) {
            result[i][j] = temp.get(j - 1);
        }
        while (j < colCount) {
            result[i][j] = 0;
            j++;
        }
    }
 
    return result;
}
{% endhighlight %}



Controller 메소드 getMetricData()는 다음과 같습니다: 



{% highlight java %}
@RequestMapping(value = "/metric-graph-data", method = RequestMethod.GET)
@ResponseBody
public Object[][] getMetricData() {
    return metricService.getGraphData();
}
{% endhighlight %}

샘플 응답은 다음과 같습니다. 


```
[
    ["Time","counter.status.302","counter.status.200","counter.status.304"],
    ["2015-03-26 19:59",3,12,7],
    ["2015-03-26 20:00",0,4,1]
]
```


# 12. Conclusion

이번 포스트에서 Spring web application에 대한 기본적인 metrics를 측정할 수 있는 간단한 방법을 살펴봤습니다. 

HTTP metric들을 측정할 수 있는 더 많은 좋은 방법들이 있지만 위 예제는 복잡한 툴 없이 간단하고 쉽게 사용할 있는 방법들입니다. 

위 포스트 예제는 (github project)[https://github.com/eugenp/tutorials/tree/master/spring-security-rest-full] 에서 확인할 수 있습니다. Eclipse 기반 프로젝트로 쉽게 import하고 실행할 수 있습니다. 