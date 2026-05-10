graph TD
    Start([App Launch]) --> CheckPIN{Has PIN?}
    
    CheckPIN -- No --> MobileLogin[Mobile Login UI]
    MobileLogin --> EnterOTP[Enter OTP]
    EnterOTP --> VerifyOTP{Verify OTP<br/>Online Required}
    VerifyOTP -- Invalid --> EnterOTP
    VerifyOTP -- Valid --> SetPIN[Set PIN]
    SetPIN --> Home[Home Screen]
    
    CheckPIN -- Yes --> PINUI[PIN Login UI]
    PINUI --> VerifyPIN{Verify PIN<br/>Local Process}
    VerifyPIN -- Invalid --> PINUI
    VerifyPIN -- Valid --> Home