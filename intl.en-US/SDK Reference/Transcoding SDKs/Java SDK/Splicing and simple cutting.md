# Splicing and simple cutting {#concept_unr_kxx_y2b .concept}

1.  Create AcsClient instance.

    ```
    DefaultProfile profile = DefaultProfile.getProfile(
                    mpsRegionId,      // Region ID
                    accessKeyId,      // AccessKey ID
                    accessKeySecret); // Access Key Secret
    IAcsClient client = new DefaultAcsClient(profile);
    ```

2.  Create request and set parameters.

    ```
    SubmitJobsRequest request = new SubmitJobsRequest();
    ```

3.  Set transcoding parameters.
    -   Input

        ```
        JSONObject input = new JSONObject();
           input.put("Location", ossLocation);
           input.put("Bucket", ossBucket);
           try {
               input.put("Object", URLEncoder.encode(headObject, "utf-8"));
           } catch (UnsupportedEncodingException e) {
               throw new RuntimeException("input URL encode failed");
           }
           request.setInput(input.toJSONString());
        ```

    -   Output

        ```
        String outputOSSObject;
           try {
               outputOSSObject = URLEncoder.encode(ossOutputObject, "utf-8");
           } catch (UnsupportedEncodingException e) {
               throw new RuntimeException("output URL encode failed");
           }
           JSONObject output = new JSONObject();
           output.put("OutputObject", outputOSSObject);
           // Ouput->TemplateId
           output.put("TemplateId", templateId);
        ```

        -   Video

            ```
            JSONObject video = new JSONObject();
             video.put("Width", "1280");
             video.put("Height", "720");
             output.put("Video", video.toJSONString());
            ```

        -   MergeList

            ```
            JSONObject mergeVideo = new JSONObject();
            String mergeVideoURL;
            try {
                mergeVideoURL = String.format(
                                            "http://%s.%s.aliyuncs.com/%s",
                                            ossBucket,
                                            ossLocation,
                                            URLEncoder.encode(ossInputObject, "utf-8"));
            } catch (UnsupportedEncodingException e) {
                throw new RuntimeException("mergeVideoURL encode failed");
            }
            mergeVideo.put("MergeURL", mergeVideoURL);
            JSONObject mergeTail = new JSONObject();
            String mergeTailURL;
            try {
                mergeTailURL = String.format(
                                            "http://%s.%s.aliyuncs.com/%s",
                                            ossBucket,
                                            ossLocation,
                                            URLEncoder.encode(tailObject, "utf-8"));
            } catch (UnsupportedEncodingException e) {
                throw new RuntimeException("mergeTailURL encode failed");
            }
            mergeTail.put("MergeURL", mergeTailURL);
            JSONArray mergeList = new JSONArray();
            mergeList.add(mergeVideo);
            mergeList.add(mergeTail);
            output.put("MergeList", mergeList.toJSONString());
            ```

4.  Initiate API request and display returned value.

    ```
    SubmitJobsResponse response;
    response = client.getAcsResponse(request);
    System.out.println("RequestId is:"+response.getRequestId());
    if (response.getJobResultList().get(0).getSuccess()) {
        System.out.println("JobId is:" + response.getJobResultList().get(0).getJob().getJobId());
    } else {
        System.out.println("SubmitJobs Failed code:" + response.getJobResultList().get(0).getCode() +
                           " message:" + response.getJobResultList().get(0).getMessage());
    }
    ```


Full codes

```
package com.aliyun.mts;
import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.aliyuncs.profile.DefaultProfile;
import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.IAcsClient;
import com.aliyuncs.exceptions.ClientException;
import com.aliyuncs.exceptions.ServerException;
import com.aliyuncs.mts.model.v20140618.*;
public class Merge {
    private static String accessKeyId = "xxx";
    private static String accessKeySecret = "xxx";
    private static String mpsRegionId = "cn-hangzhou";
    private static String pipelineId = "xxx";
    private static String templateId = "S00000001-200030";
    private static String ossLocation = "oss-cn-hangzhou";
    private static String ossBucket = "xxx";
    private static String ossInputObject = "input.mp4";
    private static String ossOutputObject = "output.mp4";
    private static String headObject = "head.mp4";
    private static String tailObject = "tail.mp4";
    public static void main(String[] args) {
        // DefaultAcsClient
        DefaultProfile profile = DefaultProfile.getProfile(
                mpsRegionId,      // Region ID
                accessKeyId,      // AccessKey ID
                accessKeySecret); // Access Key Secret
        IAcsClient client = new DefaultAcsClient(profile);
        // request
        SubmitJobsRequest request = new SubmitJobsRequest();
        // Input
        JSONObject input = new JSONObject();
        input.put("Location", ossLocation);
        input.put("Bucket", ossBucket);
        try {
            input.put("Object", URLEncoder.encode(headObject, "utf-8"));
        } catch (UnsupportedEncodingException e) {
            throw new RuntimeException("input URL encode failed");
        }
        request.setInput(input.toJSONString());
        // Output
        String outputOSSObject;
        try {
            outputOSSObject = URLEncoder.encode(ossOutputObject, "utf-8");
        } catch (UnsupportedEncodingException e) {
            throw new RuntimeException("output URL encode failed");
        }
        JSONObject output = new JSONObject();
        output.put("OutputObject", outputOSSObject);
        // Ouput->TemplateId
        output.put("TemplateId", templateId);
        // Ouput->Video
        JSONObject video = new JSONObject();
        video.put("Width", "1280");
        video.put("Height", "720");
        output.put("Video", video.toJSONString());
        // Output->MergeList
        JSONObject mergeVideo = new JSONObject();
        String mergeVideoURL;
        try {
            mergeVideoURL = String.format(
                                        "http://%s.%s.aliyuncs.com/%s",
                                        ossBucket,
                                        ossLocation,
                                        URLEncoder.encode(ossInputObject, "utf-8"));
        } catch (UnsupportedEncodingException e) {
            throw new RuntimeException("mergeVideoURL encode failed");
        }
        mergeVideo.put("MergeURL", mergeVideoURL);
        JSONObject mergeTail = new JSONObject();
        String mergeTailURL;
        try {
            mergeTailURL = String.format(
                                        "http://%s.%s.aliyuncs.com/%s",
                                        ossBucket,
                                        ossLocation,
                                        URLEncoder.encode(tailObject, "utf-8"));
        } catch (UnsupportedEncodingException e) {
            throw new RuntimeException("mergeTailURL encode failed");
        }
        mergeTail.put("MergeURL", mergeTailURL);
        JSONArray mergeList = new JSONArray();
        mergeList.add(mergeVideo);
        mergeList.add(mergeTail);
        output.put("MergeList", mergeList.toJSONString());
        // Outputs
        JSONArray outputs = new JSONArray();
        outputs.add(output);
        request.setOutputs(outputs.toJSONString());
        request.setOutputBucket(ossBucket);
        request.setOutputLocation(ossLocation);
        // PipelineId
        request.setPipelineId(pipelineId);
        // call api
        SubmitJobsResponse response;
        try {
            response = client.getAcsResponse(request);
            System.out.println("RequestId is:"+response.getRequestId());
            if (response.getJobResultList().get(0).getSuccess()) {
                System.out.println("JobId is:" + response.getJobResultList().get(0).getJob().getJobId());
            } else {
                System.out.println("SubmitJobs Failed code:" + response.getJobResultList().get(0).getCode() +
                                   " message:" + response.getJobResultList().get(0).getMessage());
            }
        } catch (ServerException e) {
            e.printStackTrace();
        } catch (ClientException e) {
            e.printStackTrace();
        }
    }
}
```

