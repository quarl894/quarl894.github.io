---
layout: post
# title:  "네이버 TTS 사용법(Clova Speech Synthesis)"
date:   2018-02-03 21:41:58 +0900
categories: android
---

지난 주 Unithon 해커톤을 나갔습니다.

2박3일로 App 개발을 하게 되었는데 그 과정에서 TTS을 사용하게 되었습니다.

TTS는 Naver API를 사용하였고 텍스트를 음성으로 변환해서 음성을 들려주는 서비스인데 지난 프로젝트에서 STT를 해본 경험이 있어 쉽게 할 수 있었습니다.

네이버에서 API를 받는 과정은 생략하고 코드 부분만 설명하겠습니다.

먼저 구현예제는 Naver Developers에 있습니다.

## APITTS Class 생성

```java
public class APITTS {
    private static String TAG = "APIExamTTS";
    public static void main(String[] args) {
        String clientId = " ";//애플리케이션 클라이언트 아이디값";
        String clientSecret = " ";//애플리케이션 클라이언트 시크릿값";
        try {
            String text = URLEncoder.encode(args[0], "UTF-8"); // 13자
            String apiURL = "https://openapi.naver.com/v1/voice/tts.bin";
            URL url = new URL(apiURL);
            HttpURLConnection con = (HttpURLConnection) url.openConnection();
            con.setRequestMethod("POST");
            con.setRequestProperty("X-Naver-Client-Id", clientId);
            con.setRequestProperty("X-Naver-Client-Secret", clientSecret);
            // post request
            String postParams = "speaker=jinho&speed=0&text=" + text;
            con.setDoOutput(true);
            con.setDoInput(true);
            DataOutputStream wr = new DataOutputStream(con.getOutputStream());
            Log.d(TAG, String.valueOf(wr));
            wr.writeBytes(postParams);
            wr.flush();
            wr.close();
            int responseCode = con.getResponseCode();
            BufferedReader br;
            if(responseCode==200) { // 정상 호출
                InputStream is = con.getInputStream();
                int read = 0;
                byte[] bytes = new byte[1024];
                //폴더를 만들어 줘야함. 없으면 새로 생성하도록 해야 한다. 예제는 Naver 폴더를 만들어서 함.
                File dir = new File(Environment.getExternalStorageDirectory()+"/", "Naver");
                if(!dir.exists()){
                    dir.mkdirs();
                }
                // 랜덤한 이름으로 mp3 파일 생성
                //String tempname = Long.valueOf(new Date().getTime()).toString();
                String tempname = "naverttstemp"; //하나의 파일명으로 덮어쓰기 하자.
                File f = new File(Environment.getExternalStorageDirectory() + File.separator + "Naver/" + tempname + ".mp3");
                f.createNewFile();
                OutputStream outputStream = new FileOutputStream(f);
                while ((read =is.read(bytes)) != -1) {
                    outputStream.write(bytes, 0, read);
                }
                is.close();

                //여기서 바로 재생하도록 하자.
                String Path_to_file = Environment.getExternalStorageDirectory()+File.separator+"Naver/"+tempname+".mp3";
                MediaPlayer audioPlay = new MediaPlayer();
                audioPlay.setDataSource(Path_to_file);
                audioPlay.prepare(); // 파일 재생 과정
                audioPlay.start();
            } else {  // 에러 발생
                br = new BufferedReader(new InputStreamReader(con.getErrorStream()));
                String inputLine;
                StringBuffer response = new StringBuffer();
                while ((inputLine = br.readLine()) != null) {
                    response.append(inputLine);
                }
                br.close();
            }
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
```

## 음성을 적용시킬 Activity

Activity에서는 Text를 임의로 저장된 값을 TTS를 통해 MP3를 받아 실행시키는 방식이다.

(Unithon에서는 서버에서 텍스트를 받아오는 방식으로 함)

Text를 받는 과정이나 서버에서 텍스트를 불러오는 과정은 개발자가 구현해주면 된다.

``` java
public class DetailActivity extends AppCompatActivity{
    private NaverTTSTask mNaverTTSTask;
    String[] mTextString;
    Button btTTS, btReset;
    ArrayList<String> ttsList;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_detail);

        btTTS = (Button) findViewById(R.id.bt_tts);
        btReset = (Button) findViewById(R.id.bt_reset);

		ttsList = new ArrayList<>();

        ttsList.add("오늘은 비를 맞았어요");
        ttsList.add("저는 거의 다 익었어요");
        ttsList.add("오늘은 광합성 짱이에요");

        //버튼 클릭이벤트 - 클릭하면 저장된 글자를 네이버에 보낸다.
        btTTS.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                int random = (int)(Math.random()*3);
                String result = ttsList.get(random);

                //사용자가 입력한 텍스트를 이 배열변수에 담는다.
                if(result.length()>0){ //한글자 이상 1
                    mTextString = new String[]{result};

                    //AsyncTask 실행
                    mNaverTTSTask = new NaverTTSTask();
                    mNaverTTSTask.execute(mTextString);
                } else {
                    Toast.makeText(DetailActivity.this, "텍스트가 없음.", Toast.LENGTH_SHORT).show();
                    return;
                }
            }
        });

        //리셋버튼
        btReset.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                etText.setText("");
                etText.setHint("텍스트를 입력하세요.");
            }
        });

		// MP3를 저장하기 때문에 저장공간 Permission을 줘야함.
        PermissionListener permissionlistener = new PermissionListener() {
            @Override
            public void onPermissionGranted() {
                Toast.makeText(DetailActivity.this, "Permission Granted", Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onPermissionDenied(ArrayList<String> deniedPermissions) {
                Toast.makeText(DetailActivity.this, "권한이 거부되었습니다. 권한거부시 앱기능 일부분을 사용하실수 없습니다.", Toast.LENGTH_SHORT).show();
            }
        };
        TedPermission.with(DetailActivity.this)
                .setPermissionListener(permissionlistener)
                .setDeniedMessage("If you reject permission,you can not use this service\n\nPlease turn on permissions at [Setting] > [Permission]")
                .setPermissions(Manifest.permission.WRITE_EXTERNAL_STORAGE)
                .check();
    }
    
    private class NaverTTSTask extends AsyncTask<String[], Void, String> {
        @Override
        protected String doInBackground(String[]... strings) {
            //여기서 서버에 요청
            APITTS.main(mTextString);
            return null;
        }

        @Override
        protected void onPostExecute(String result) {
            super.onPostExecute(result);

        }
    }
}
```
[jekyll-gh]:   https://github.com/quarl894
