# howToUseFirebase

### ※나의 Android 프로젝트에 FireBase 추가  

<ol>
  <li> firebase사이트에서 나만의 프로젝트를 만든다.(프로젝트 추가하기)
  <li> 앱 추가하기를 진행한다.(앱 추가) 그 후, 플랫폼을 선택한다.
  <li> 가이드 라인에 맞추어 프로젝트에 적절히 json 파일 및 gradle을 핸들링 한다.
  <li> 가이드라인에 맞추어 설정을 마쳤다면 아래 링크를 참고하여 적절한 조치를 취한다.
</ol>  
<a href=https://firebase.google.com/docs/android/setup>Firebase 추가 가이드 라인</a>  
  
  
### ※구글인증을 통한 Firebase인증
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
  
  
### ※Email, Password를 통한 회원가입 및 로그인
<ul>
  <a href=https://firebase.google.cn/docs/auth/android/start><li>로그인 및 비밀번호 인증 가이드라인 참고</li></a>
</ul>
  
```
public class MainActivity extends AppCompatActivity {
    private EditText email,pass; //email, password 에딧텍스트
    private Button sign; // 로그인 및 회원가입 버튼
    private Button logout;
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
        logout=findViewById(R.id.logout);
        email=findViewById(R.id.emailtext);
        pass=findViewById(R.id.passwordtext);
        sign=findViewById(R.id.signup);

        mAuth = FirebaseAuth.getInstance(); //파이어베이스 인증을 위한 객체생성

        logout.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                signOut();
            }
        });
        sign.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                signInUser(email.getText().toString(),pass.getText().toString());
            }
        });
    }
    public void createNewAccount(String email,String password){ //회원가입 메소드
        mAuth.createUserWithEmailAndPassword(email, password)
                .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if (task.isSuccessful()) { // 인증이 성공시,
                            FirebaseUser user = mAuth.getCurrentUser(); //인증받은 user의 객체
                            Toast.makeText(MainActivity.this,"회원가입 성공",Toast.LENGTH_SHORT).show();
                            //DoSomething(user);

                        }
                        else {// 인증이 실패시,
                           //Something else...
                        }
                    }
                });
    }
    public void signInUser(String email,String password){//기존 사용자의 로그인 메소드
        final String Email=email;
        final String Password=password;
        mAuth.signInWithEmailAndPassword(email, password)
                .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if (task.isSuccessful()) { //인증 성공시,
                            FirebaseUser user = mAuth.getCurrentUser();
                            Toast.makeText(MainActivity.this,user.getEmail(),Toast.LENGTH_SHORT).show();
                            //DoSomething(user);
                        }
                        else { //인증 실패시,
                            createNewAccount(Email,Password); //기존 로그인 사용자가 아니라면 회원가입을 한다.
                        }
                    }
                });
    }
    private void signOut() {
        // 파이어베이스 로그아웃
        mAuth.signOut();
        Toast.makeText(MainActivity.this,"로그아웃 완료",Toast.LENGTH_SHORT).show();
    }
}
```
  
  
### ※Auth Listener  
로그인된 상태 또는 로그아웃된 상태에 따른 리스너를 설정해줄수 있다.  
#### 리스너 객체 생성(변수 선언)  
```
private FirebaseAuth.AuthStateListener authStateListener; //로그인 상태변화에 따른 리스너
```
  
  
#### 리스너 설정
```
@Override
    public void onStart() {
        super.onStart();
        FirebaseUser currentUser = mAuth.getCurrentUser();
        mAuth.addAuthStateListener(authStateListener);

        /*if(currentUser!=null)
            Toast.makeText(MainActivity.this,currentUser.getEmail(),Toast.LENGTH_SHORT).show();*/
            //리스너의 정의로 이것은 더는 필요가 없어질것입니다.

    }

    @Override
    protected void onStop() {
        super.onStop();
        if(authStateListener!=null) //리스너를 해체합니다.
            mAuth.removeAuthStateListener(authStateListener);
    }
```
  
  
#### 리스너 이벤트 정의
```
 authStateListener=new FirebaseAuth.AuthStateListener() {
            @Override
            public void onAuthStateChanged(@NonNull FirebaseAuth firebaseAuth) {
                FirebaseUser user=firebaseAuth.getCurrentUser();
                if(user!=null){
                    //DoSomething_login
                }
                else{
                    //DoSomething_logout
                }
            }
        };
```
  
  
### ※로그인 후, 새로운 Acticity에서의 Auth Control  
새로운 액티비티에서도 로그인 창에서 사용했던 Auth를 컨트롤 할 수 있다.
#### 객체 생성(변수 선언)
```
private FirebaseAuth auth; //Auth객체를 그대로 사용할 변수
```
  
  
#### FirebaseAuth 객체 가져오기
```
auth = FirebaseAuth.getInstance(); //같은 객체
```
