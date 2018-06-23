# youngkid

package spider.first_spider;

import org.apache.http.HttpEntity;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

import java.util.ArrayList;
import java.util.List;

public class FirstSpider {

    //定义登录绝对路径常量
    public static String base_path = "http://127.0.0.1:8080/springmvc-second/";
    //登陆地址
    public static String login_path = "user/login.do";
    //商品页面
    public static  String item_path = "queryItem.do";

    public static void main(String[] args) throws Exception {

        //1.获取html页面数据，自己定义一个方法
        String html = getProduct();
//        System.out.println(html);

        //2.解析html数据
        parseHtml(html);

    }


    public static String getProduct() throws Exception{

        String html = null;

        //创建HttpClient对象
        CloseableHttpClient httpClient = HttpClients.createDefault();

        //创建HttpPost对象
        HttpPost post = new HttpPost(base_path+login_path);

        /**
         * 获取请求头和请求提信息
         * User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36
         */
        post.addHeader("User-Agent","Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36");
        List<BasicNameValuePair> bnvp = new ArrayList<BasicNameValuePair>();
        bnvp.add(new BasicNameValuePair("username","xiaom"));
        bnvp.add(new BasicNameValuePair("userpwd","123"));
        post.setEntity(new UrlEncodedFormEntity(bnvp));

        //执行请求
        CloseableHttpResponse response = httpClient.execute(post);

        //获取响应码状态
        int statusCode = response.getStatusLine().getStatusCode();
//        System.out.println(statusCode);

        if(statusCode == 302){
            //创建HttpGet对象
            HttpGet get = new HttpGet(base_path+item_path);
            /**
             * 设置请求头信息
             * User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36
             * login.do	queryItem.do
             */
            get.setHeader("User-Agent","Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36");

            //执行请求
            CloseableHttpResponse response1 = httpClient.execute(get);

            //获取响应码状态
            int statusCode1 = response1.getStatusLine().getStatusCode();
//            System.out.println(statusCode1);

            if(statusCode1 == 200){
                //打印页面数据
                HttpEntity entity = response1.getEntity();
                html = EntityUtils.toString(entity, "UTF-8");
            }
        }
        return html;
    }

    private static void parseHtml(String html) throws Exception {
        //创建document对象
//        Document document = Jsoup.connect("base_path+item_path").get();
        Document document = Jsoup.parse(html);

        //获取表单对象from(第一个from)
        Elements forms = document.select("form");
        Element form1 = forms.first();
//        System.out.println(form1);

        //获取第二个table
        Elements tables = form1.select("table");
        Element table = tables.get(1);
//        System.out.println(table);

        //获取table中的tr，并且去除第一个tr
        Elements table_trs = table.select("table tr");
        table_trs.remove(0);
//        System.out.println(table_trs);

        //获取tr中的td的值
        for (Element tr : table_trs){
            //获取tr下的td
            Elements tds = tr.select("td");
            //商品ID
            String id = tds.first().select("input").attr("value");
            System.out.println("商品ID:"+id);
            //商品名称
            String name = tds.get(1).text();
            System.out.println("商品名称："+name);
            //商品价格
            String price = tds.get(2).text();
            System.out.println("商品价格："+price);
            //商品生产时间
            String time = tds.get(3).text();
            System.out.println("商品生产时间："+time);
            //商品描述
            String discrition = tds.get(4).text();
            System.out.println("商品描述："+discrition);

            System.out.println("--------------------------------------");

        }


    }
}
