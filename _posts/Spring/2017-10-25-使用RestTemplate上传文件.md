---
layout: blog
banana: true
background-image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1509503652&di=cf9ab469d47904d5a9ab910f3134110f&imgtype=jpg&er=1&src=http%3A%2F%2Fwww.sz886.com%2Feditor%2Fimage%2F20170507%2F20170507175839_7806.png
category: Spring
title: 使用RestTemplate上传文件
tags:
- RestTemplate
- 上传文件
date:   2017-10-25 10:25:54
---

最近在用Spring Cloud，搭建微服务应用，其中一个微服务是把文件上传到七牛，其他的文件上传都是通过他。但是在使用Fegin调用该服务的接口的时候，一直有问题，恩--------先用RestTemplate试试


#步骤

#1、声明对象

```
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
```

#2、发送请求


```
    @Autowired
    private RestTemplate rest;
    public void ExceptInfoForRestTemplate(String excelTitle, List<String[]> arrayList) throws
            ParseException {

        String fileLocal="E://xxccccccccc"
        String url = "http://xxxxxx.com/upload";
        FileSystemResource resource = new FileSystemResource(new File(fileLocal));
        MultiValueMap<String, Object> param = new LinkedMultiValueMap<>();
        param.add("file", resource);

        HttpEntity<MultiValueMap<String, Object>> httpEntity = new HttpEntity<MultiValueMap<String, Object>>(param);
        ResponseEntity<String> responseEntity = rest.exchange(url, HttpMethod.POST, httpEntity, String.class);
        System.out.println(responseEntity.getBody());
    }
```



