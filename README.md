# howToUseFirebase

### 나의 Android 프로젝트에 FireBase 추가  

<ol>
  <li> firebase사이트에서 나만의 프로젝트를 만든다.(프로젝트 추가하기)
  <li> 앱 추가하기를 진행한다.(앱 추가) 그 후, 플랫폼을 선택한다.
  <li> 가이드 라인에 맞추어 프로젝트에 적절히 json 파일 및 gradle을 핸들링 한다.
  <li> 가이드라인에 맞추어 설정을 마쳤다면 아래 링크를 참고하여 적절한 조치를 취한다.
</ol>  
<a href=https://firebase.google.com/docs/android/setup>Firebase 추가 가이드 라인</a>  
  
  
### 구글인증을 통한 Firebase인증
<ul>
  <a href=https://firebase.google.com/docs/auth/android/google-signin><li>구글 인증 및 Firebase 인증 가이드라인 참고</li></a>
</ul>
  
```
public class MainActivity extends AppCompatActivity {
    private SignInButton button;
    private Button logout;
    private GoogleSignInClient mGoogleSignInClient;
    private int RC_SIGN_IN = 10;
    private FirebaseAuth mAuth;
    @Override
    public void onStart() {
        super.onStart();
        FirebaseUser currentUser = mAuth.getCurrentUser();
        if(currentUser!=null)
            Toast.makeText(MainActivity.this,currentUser.getEmail(),Toast.LENGTH_SHORT).show();

    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        button=findViewById(R.id.loginbtn);
        logout=findViewById(R.id.logout);

        GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
                .requestIdToken(getString(R.string.default_web_client_id))
                .requestEmail()
                .build(); //구글 인증을 할때의 옵션 설정

        mGoogleSignInClient = GoogleSignIn.getClient(this, gso); //구글 인증을 위한 객체생성
        mAuth = FirebaseAuth.getInstance(); //파이어베이스 인증을 위한 객체생성

        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                signIn();
            }
        });
        logout.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                signOut();
            }
        });
    }
    private void signIn() { //google plus에 요청 : 이 사람이 구글 사용자니?
        Intent signInIntent = mGoogleSignInClient.getSignInIntent(); //인증할수있는곳 intent
        startActivityForResult(signInIntent, RC_SIGN_IN);
    }
    private void signOut() {
        // 파이어베이스 로그아웃
        mAuth.signOut();

        // 구글 로그아웃
        mGoogleSignInClient.signOut().addOnCompleteListener(this,
                new OnCompleteListener<Void>() {
                    @Override
                    public void onComplete(@NonNull Task<Void> task) {
                        Toast.makeText(MainActivity.this,"로그아웃 완료",Toast.LENGTH_SHORT).show();
                    }
                }); //로그아웃에 대한 리스너 설정
    }


    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        if (requestCode == RC_SIGN_IN) { //응 구글 사용자 맞아!
            Task<GoogleSignInAccount> task = GoogleSignIn.getSignedInAccountFromIntent(data);
            try {
                // 구글인증이 성공시, 파이어베이스로 인증
                GoogleSignInAccount account = task.getResult(ApiException.class);
                firebaseAuthWithGoogle(account); //그럼 파이어베이스 인증으로 넘길께!
            } catch (ApiException e) {
                // 구글인증 실패
                Log.w("TAG", "Google sign in failed", e);
            }
        }
    }
    private void firebaseAuthWithGoogle(GoogleSignInAccount acct) {

        AuthCredential credential = GoogleAuthProvider.getCredential(acct.getIdToken(), null); //그 사람에 대한 정보를 받고,
        mAuth.signInWithCredential(credential)
                .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() { //해당 인증에대한 리스너 생성
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if (task.isSuccessful()) { //파이어베이스 인증이 성공
                            FirebaseUser user = mAuth.getCurrentUser(); //해당 유저객체
                            Toast.makeText(MainActivity.this,user.getEmail(),Toast.LENGTH_SHORT).show(); //사용자 이메일을 토스트 메세지로
                            
                            //DoSomething(user)

                        } else { //인증실패
                            //Something else...
                        }
                    }
                });
    }
}
```
