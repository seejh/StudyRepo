
# JWT
JWT는 토큰 기반 "인증" 방식 중 하나이며 URL, HTTP 헤더 또는 바디를 통해 쉽게 전송된다. <br/>

## 동작 방식
JWT의 인증 시스템은 대부분 아래와 같으며 일반적으로 세 가지 주요 구성 요소가 있다. <br/>
일반적으로 권한 부여 서버(로그인하여 인증 받는 곳)와 리소스 서버(요청을 할 곳)가 분리되어 있으며 권한 부여 서버로 인증하고 <br/>
토큰을 받은 뒤 앞으로의 리소스 서버로의 요청에 해당 토큰을 첨부함으로써 본인의 자격을 증명한다 <br/>
### 클라이언트
보호된 리소스에 액세스를 요청하는 엔티티 (요청하는 자(사용자), 요청하는 자의 브라우저 또는 디바이스) <br/>
### 권한 부여 서버
클라이언트를 인증하고 JWT를 발행하는 서버 (사용자의 로그인을 대행하고 JWT를 발행해 인가하는 곳) <br/>
### 리소스 서버
보호된 리소스를 소유한 곳 (사용자가 접근하고 싶은 리소스가 있는 곳) <br/>

### JWT 동작 도식화 
![image](https://github.com/user-attachments/assets/70cdb3d3-2781-426e-a9ad-6b74f95c0276)
1 클라이언트(사용자)가 권한 부여 서버에 자격증명(ID, PW)을 제출 (=로그인) <br/>
2 권한 부여 서버는 두 가지 유형의 토큰을 생성(리프레시 토큰 사용 방식)하고 클라이언트에 토큰을 보낸다. <br/>
3 클라이언트에서 받은 토큰을 저장(로컬 저장소, 세션 저장소 또는 쿠키) <br/>
4 이후 클라이언트에서 리소스 서버의 보호된 리소스에 접근 요청 시마다, 토큰 첨부하여 요청 <br/>
&emsp;&emsp; HTTP 권한 부여 헤더에 액세스 토큰을 첨부 <br/>
&emsp;&emsp; 토큰은 일반적으로 Bearer 스키마인 Authorization:Bearaer<access_token> 형식 <br/>
5 리소스 서버에서 요청을 수신 후 액세스 토큰 유효성 검사 (토큰의 서명, 만료 시간, 발급자 등을 확인하는 작업 포함) <br/>
5-1 토큰 유효 => 요청한 리소스로 클라이언트에 응답 <br/>
5-2 토큰 유효 X => 액세스 거부(일반적으로 401 권한 없음), 이를 받은 클라이언트에서 리프레시 토큰을 리소스 서버로 전송, <br/>
6 새 액세스 토큰과 새 리프레시 토큰을 획득 후 다시 요청 <br/>

<hr/>

# 실습
ASP.NET Core Web API 프로젝트, JWT의 구성 요소는 아래와 같이 처리 <br/>
인증 서버 - 클라이언트 자격 증명을 검사하고 액세스 토큰 및 리프레시 토큰을 생성하는 인증 서버 역할의 엔드포인트 제공 <br/>
리소스 서버 - 보호된 리소스를 소유하고 사용자의 요청을 받을 리소스 서버 역할의 엔드포인트 제공 <br/>
클라이언트 - Postman 사용 <br/>

## 사용 패키지
Microsoft.EntityFrameworkCore.SqlServer - EFCore, MSSQL 사용<br/>
Microsoft.EntityFrameworkCore.Tools - EFCore 명령어 사용<br/>
Microsoft.AspNetCore.Authentication.JwtBearer - JWT 사용<br/>
BCrypt.Net-Next - 암호화 알고리즘 사용<br/>

## 엔티티
### SigningKey
RSA 키를 동적으로 관리하기 위해 키 정보를 보유하는 SigningKey 엔티티를 사용한다. <br/>
```c#
using System.ComponentModel.DataAnnotations;

namespace JWTAuthServer.Models
{
    public class SigningKey
    {
        [Key]
        public int Id { get; set; }

        // 고유 식별자
        [Required]
        [MaxLength(100)]
        public string KeyId { get; set; }

        // 개인키, 공개키(XML 또는 PEM 형식)
        [Required]
        public string PrivateKey { get; set; }
        [Required]
        public string PublicKey { get; set; }

        // 키 활성 상태, 키 생성 시간, 키 만료 시간
        [Required]
        public bool IsActive { get; set; }
        [Required]
        public DateTime CreatedAt { get; set; }
        [Required]
        public DateTime ExpiresAt { get; set; }
    }
}
```

## 데이터 전송 객체(DTO) 생성
DTO(Data Transfer Objects)는 클라이언트와 서버 간에 전송되는 데이터를 캡슐화하는데 사용되어 필요한 정보만 노출되도록 한다. <br/>

### RegisterDTO
사용자 등록에 필요한 정보를 캡쳐 <br/>
```c#
using System.ComponentModel.DataAnnotations;
namespace JWTAuthServer.DTOs
{
    public class RegisterDTO
    {
        [Required(ErrorMessage = "First name is required.")]
        [MaxLength(50, ErrorMessage = "First name must be less than or equal to 50 characters.")]
        public string Firstname { get; set; }
        [MaxLength(50, ErrorMessage = "Last name must be less than or equal to 50 characters.")]
        public string Lastname { get; set; }
        [Required(ErrorMessage = "Email is required.")]
        [EmailAddress(ErrorMessage = "Invalid email address.")]
        [MaxLength(100, ErrorMessage = "Email must be less than or equal to 100 characters.")]
        public string Email { get; set; }
        [Required(ErrorMessage = "Password is required.")]
        [MinLength(6, ErrorMessage = "Password must be at least 6 characters long.")]
        [MaxLength(100, ErrorMessage = "Password must be less than or equal to 100 characters.")]
        public string Password { get; set; }
    }
}
```

### LoginDTO
클라이언트가 토큰을 요청하려고 할 때 사용하는 클라이언트 자격(토큰을 요청하는 애플리케이션을 알 수 있도록) 증명, <br/>
사용자 자격 증명(어떤 사용자가 로그인하는지 알 수 있도록) 양식 <br/>

```c#
using System.ComponentModel.DataAnnotations;
namespace JWTAuthServer.DTOs
{
    public class LoginDTO
    {
        // Email input from the user during login.
        [EmailAddress]
        [Required(ErrorMessage = "Email is required.")]
        [MaxLength(100, ErrorMessage = "Email must be less than or equal to 100 characters.")]
        public string Email { get; set; }
        // Password input from the user during login.
        [Required(ErrorMessage = "Password is required.")]
        [MinLength(6, ErrorMessage = "Password must be at least 6 characters long.")]
        [MaxLength(100, ErrorMessage = "Password must be less than or equal to 100 characters.")]
        public string Password { get; set; }
        [Required(ErrorMessage = "ClientId is required.")]
        public string ClientId { get; set; }
    }
}
```
## 서비스(백그라운드)
ASP.NET Core의 백그라운드 서비스는 들어오는 HTTP 요청과 독립적으로 백그라운드에서 작업을 실행하는 장기 실행 서비스이다 <br/>
BackgroundService 기본 클래스를 상속 받는 클래스를 만들거나 IHostedService 인터페이스를 사용하여 구현한다. <br/>

### KeyRotationService
주기적으로 새 RSA 키 쌍을 생성하고 이전 키 쌍을 비활성화하여 보안을 강화하는 백그라운드 서비스 <br/>
```c#
using JWTAuthServer.Data;
using JWTAuthServer.Models;
using Microsoft.EntityFrameworkCore;
using System.Security.Cryptography;

namespace JWTAuthServer.Services
{
    public class KeyRotationService : BackgroundService
    {
        // Service provider is used to create a scoped service lifetime.
        private readonly IServiceProvider _serviceProvider; // 
        private readonly TimeSpan _rotationInterval = TimeSpan.FromDays(7); // 키 변경 주기, 여기선 일주일 단위
        public KeyRotationService(IServiceProvider serviceProvider) {
            _serviceProvider = serviceProvider;
        }

        // 백그라운드 서비스가 시작될 때 실행
        // 이 백그라운드 서비스의 작업 로직이 상주하는 핵심 메서드, BackgroundService 기본 클래스의 재정의 메서드
        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            // 일반적으로 ExecuteAsync에는 무기한 루프가 있으며(프로그램 정지시까지)
            // 팽ㅍ챙한 루프를 방지하기 위해 중간에 지연을 두고 수행한다.
            while (!stoppingToken.IsCancellationRequested)
            {
                // 일정 간격으로 키 변경 로직 수행
                await RotateKeysAsync();
                await Task.Delay(_rotationInterval, stoppingToken);
            }
        }

        // 키 변경 로직
        private async Task RotateKeysAsync()
        {
            // Create a new service scope for dependency injection.
            using var scope = _serviceProvider.CreateScope();

            // 서비스 공급자로부터 DbContext를 가져온다
            var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();

            // DB에서 현재 활성 상태인 서명키를 검색한다.
            var activeKey = await context.SigningKeys.FirstOrDefaultAsync(k => k.IsActive);

            // 활성된 키가 없거나 곧 만료되는지 확인
            if (activeKey == null || activeKey.ExpiresAt <= DateTime.UtcNow.AddDays(10))
            {
                // 활성키가 있으면 비활성으로 변경
                if (activeKey != null)
                {
                    activeKey.IsActive = false;
                    context.SigningKeys.Update(activeKey);
                }

                // 새 RSA 키 쌍 생성
                using var rsa = RSA.Create(2048);

                // 개인키, 공개키를 Base64로 인코딩된 문자열로 추출
                var privateKey = Convert.ToBase64String(rsa.ExportRSAPrivateKey());
                var publicKey = Convert.ToBase64String(rsa.ExportRSAPublicKey());

                // 새 키에 대한 고유 식별자 생성
                var newKeyId = Guid.NewGuid().ToString();

                // 새 RSA 키 세부 정보를 사용하여 새 서명키엔티티 생성
                var newKey = new SigningKey
                {
                    KeyId = newKeyId,
                    PrivateKey = privateKey,
                    PublicKey = publicKey,
                    IsActive = true,
                    CreatedAt = DateTime.UtcNow,
                    ExpiresAt = DateTime.UtcNow.AddYears(1)
                };

                // DB에 새로운 서명키 추가
                await context.SigningKeys.AddAsync(newKey);
                await context.SaveChangesAsync();
            }
        }
    }
}
```

## 컨트롤러
### 사용자 컨트롤러
```c#
using JWTAuthServer.Data;
using JWTAuthServer.DTOs;
using JWTAuthServer.Models;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace JWTAuthServer.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class UsersController : ControllerBase
    {
        private readonly ApplicationDbContext _context;
        public UsersController(ApplicationDbContext context) // 생성자에서 ~DbContext를 주입
        {
            _context = context; 
        }

        // 새 사용자 등록
        [HttpPost("Register")]
        public async Task<IActionResult> Register([FromBody] RegisterDTO registerDto)
        {
            // 모델 유효성 검사
            if (!ModelState.IsValid) { return BadRequest(ModelState); }

            // 해당 이메일이 이미 존재하는지 확인
            var existingUser = await _context.Users.FirstOrDefaultAsync(u => u.Email.ToLower() == registerDto.Email.ToLower());
            if (existingUser != null) { return Conflict(new { message = "Email is already registered." }); }

            // BCrypt를 사용해서 패스워드를 해시
            string hashedPassword = BCrypt.Net.BCrypt.HashPassword(registerDto.Password);

            // 유저 엔티티 생성, DB에 추가
            var newUser = new User
            {
                Firstname = registerDto.Firstname,
                Lastname = registerDto.Lastname,
                Email = registerDto.Email,
                Password = hashedPassword
            };
            _context.Users.Add(newUser);
            await _context.SaveChangesAsync();

            // 선택사항, 유저에게 역할을 할당, 여기서는 새로운 유저에게 User라는 역할을 할당
            var userRole = await _context.Roles.FirstOrDefaultAsync(r => r.Name == "User");
            if (userRole != null)
            {
                var newUserRole = new UserRole
                {
                    UserId = newUser.Id,
                    RoleId = userRole.Id
                };
                _context.UserRoles.Add(newUserRole);
                await _context.SaveChangesAsync();
            }

            // 201 응답(생성 성공)
            return CreatedAtAction(nameof(GetProfile), new { id = newUser.Id }, new { message = "User registered successfully." });
        }

        // 인증된 유저의 프로필을 가져온다.
        [HttpGet("GetProfile")]
        [Authorize]
        public async Task<IActionResult> GetProfile()
        {
            // Extract the user's email from the JWT token claims.
            var emailClaim = User.Claims.FirstOrDefault(c => c.Type == System.Security.Claims.ClaimTypes.Email);
            if (emailClaim == null)
            {
                return Unauthorized(new { message = "Invalid token: Email claim missing." });
            }
            string userEmail = emailClaim.Value;
            // Retrieve the user from the database, including roles.
            var user = await _context.Users
                .Include(u => u.UserRoles)
                    .ThenInclude(ur => ur.Role)
                .FirstOrDefaultAsync(u => u.Email.ToLower() == userEmail.ToLower());
            if (user == null)
            {
                return NotFound(new { message = "User not found." });
            }
            // Map the user entity to ProfileDTO.
            var profile = new ProfileDTO
            {
                Id = user.Id,
                Email = user.Email,
                Firstname = user.Firstname,
                Lastname = user.Lastname,
                Roles = user.UserRoles.Select(ur => ur.Role.Name).ToList()
            };

            return Ok(profile);
        }

        // 인증된 유저의 프로필 업데이트
        [HttpPut("UpdateProfile")]
        [Authorize]
        public async Task<IActionResult> UpdateProfile([FromBody] UpdateProfileDTO updateDto)
        {
            // Validate the incoming model.
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }
            // Extract the user's email from the JWT token claims.
            var emailClaim = User.Claims.FirstOrDefault(c => c.Type == System.Security.Claims.ClaimTypes.Email);
            if (emailClaim == null)
            {
                return Unauthorized(new { message = "Invalid token: Email claim missing." });
            }
            string userEmail = emailClaim.Value;
            // Retrieve the user from the database.
            var user = await _context.Users
                .FirstOrDefaultAsync(u => u.Email.ToLower() == userEmail.ToLower());
            if (user == null)
            {
                return NotFound(new { message = "User not found." });
            }
            // Update fields if provided.
            if (!string.IsNullOrEmpty(updateDto.Firstname))
            {
                user.Firstname = updateDto.Firstname;
            }
            if (!string.IsNullOrEmpty(updateDto.Lastname))
            {
                user.Lastname = updateDto.Lastname;
            }
            if (!string.IsNullOrEmpty(updateDto.Email))
            {
                // Check if the new email is already taken by another user.
                var emailExists = await _context.Users
                    .AnyAsync(u => u.Email.ToLower() == updateDto.Email.ToLower() && u.Id != user.Id);
                if (emailExists)
                {
                    return Conflict(new { message = "Email is already in use by another account." });
                }
                user.Email = updateDto.Email;
            }
            if (!string.IsNullOrEmpty(updateDto.Password))
            {
                // Hash the new password before storing.
                string hashedPassword = BCrypt.Net.BCrypt.HashPassword(updateDto.Password);
                user.Password = hashedPassword;
            }
            // Save the changes to the database.
            _context.Users.Update(user);
            await _context.SaveChangesAsync();
            return Ok(new { message = "Profile updated successfully." });
        }
    }
}
```

### 인증 컨트롤러
API 컨트롤러, 사용자를 인증하고 JWT 토큰을 발행한다. <br/>
```c#
using JWTAuthServer.Data;
using JWTAuthServer.DTOs;
using JWTAuthServer.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Security.Cryptography;

namespace JWTAuthServer.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class AuthController : ControllerBase
    {
        private readonly IConfiguration _configuration;
        private readonly ApplicationDbContext _context; 
        public AuthController(IConfiguration configuration, ApplicationDbContext context)
        {
            _configuration = configuration;
            _context = context; 
        }

        // 로그인 인증 요청
        // POST, 'api/Auth/Login'
        [HttpPost("Login")]
        public async Task<IActionResult> Login([FromBody] LoginDTO loginDto)
        {
            // LoginDTO의 애노테이션을 기반으로 들어오는 모델 검증
            if (!ModelState.IsValid) { return BadRequest(ModelState); }

            // 클라이언트 테이블에 해당하는 데이터가 있는지 검색
            var client = _context.Clients.FirstOrDefault(c => c.ClientId == loginDto.ClientId);
            if (client == null) { return Unauthorized("Invalid client credentials."); }

            // DB에서 이메일이 일치하는 사용자를 찾는다 (대소문자 구분)
            // 
            var user = await _context.Users.Include(u => u.UserRoles)
                    .ThenInclude(ur => ur.Role)
                    .FirstOrDefaultAsync(u => u.Email.ToLower() == loginDto.Email.ToLower());
            if (user == null) { return Unauthorized("Invalid credentials."); }

            // BCrypt를 사용하여 제공된 패스워드를 해시되어 저장된 패스워드로 검증한다.
            bool isPasswordValid = BCrypt.Net.BCrypt.Verify(loginDto.Password, user.Password);
            if (!isPasswordValid) { return Unauthorized("Invalid credentials."); }

            // 로그인 인증 성공
            // JWT 토큰 발행, 토큰 포함 200 OK 응답
            var token = GenerateJwtToken(user, client);
            return Ok(new { Token = token });
        }

        // JWT 토큰 발행, private 메서드
        private string GenerateJwtToken(User user, Client client)
        {
            // DB에서 현재 활성화된 서명키를 가져온다.
            var signingKey = _context.SigningKeys.FirstOrDefault(k => k.IsActive);
            if (signingKey == null) { throw new Exception("No active signing key available."); }

            // 서명키의 비밀키를 Base64 -> 바이트 배열로 다시 변환
            // 해당 비밀키로 RSA 보안키 생성
            var privateKeyBytes = Convert.FromBase64String(signingKey.PrivateKey);
            var rsa = RSA.Create();
            rsa.ImportRSAPrivateKey(privateKeyBytes, out _);
            var rsaSecurityKey = new RsaSecurityKey(rsa)
            {
                // 키 ID를 할당하여 JWT를 올바른 공개키와 연결
                KeyId = signingKey.KeyId 
            };

            // RSA 비밀키와 명시한 암호화 알고리즘을 사용하여 자격증명 생성
            var creds = new SigningCredentials(rsaSecurityKey, SecurityAlgorithms.RsaSha256);
            // JWT에 포함할 클레임 목록 정의
            var claims = new List<Claim>
            {
                new Claim(JwtRegisteredClaimNames.Sub, user.Id.ToString()), // 제목 클레임, 유저 ID
                new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()), // JWT ID (jti, 토큰을 구분하기 위한 고유 번호)
                new Claim(ClaimTypes.Name, user.Firstname), // 이름 클레임, 유저의 이름
                new Claim(ClaimTypes.NameIdentifier, user.Email), // 이름 식별자, 유저의 이메일로
                new Claim(ClaimTypes.Email, user.Email) // 이메일 클레임, 유저의 이메일
            };

            // Iterate through the user's roles and add each as a Role claim
            // 유저 역할 ~해서 역할 클레임으로 각각 추가한다.
            foreach (var userRole in user.UserRoles)
            {
                claims.Add(new Claim(ClaimTypes.Role, userRole.Role.Name));
            }

            // JWT 토큰 속성 정의
            var tokenDescriptor = new JwtSecurityToken(
                issuer: _configuration["Jwt:Issuer"], // 토큰 발급자, 일반적으로 앱의 URL
                audience: client.ClientURL, // 토큰 수신자, 일반적으로 클라이언트 URL
                claims: claims, // 토큰에 포함될 클레임 목록
                expires: DateTime.UtcNow.AddHours(1), // 토큰 만료 시간, 현재 1시간 후 만료 설정
                signingCredentials: creds // 토큰 서명에 사용되는 자격증명
            );

            // JWT토큰 직렬화를 위해 JWT 토큰 핸들러 생성
            // JWT 토큰을 string으로 직렬화 후 리턴
            var tokenHandler = new JwtSecurityTokenHandler();
            var token = tokenHandler.WriteToken(tokenDescriptor);
            return token;
        }
    }
}
```

## 기본 설정
### Program.cs
```c#
using JWTAuthServer.Data;
using Microsoft.EntityFrameworkCore;
using JWTAuthServer.Services;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;

namespace JWTAuthServer
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var builder = WebApplication.CreateBuilder(args);
            // 컨테이너에 컨트롤러 서비스를 추가하고 JSON 직렬화 옵션을 구성한다.
            builder.Services.AddControllers().AddJsonOptions(options =>
                {
                    options.JsonSerializerOptions.PropertyNamingPolicy = null; // CamelCase 이름 지정 비활성화
                });
            builder.Services.AddEndpointsApiExplorer(); 
            builder.Services.AddSwaggerGen();

            // EF Core 구성
            builder.Services.AddDbContext<ApplicationDbContext>(options =>
                options.UseSqlServer(builder.Configuration.GetConnectionString("EFCoreDBConnection")));

            // 백그라운드 서비스(KeyRotationService) 등록
            builder.Services.AddHostedService<KeyRotationService>();

            // 인증에 JWT Bearer token을 사용하도록 설정
            builder.Services.AddAuthentication(options =>
            {
                // This indicates the authentication scheme that will be used by default when the app attempts to authenticate a user.
                // Which authentication handler to use for verifying who the user is by default.
                // 앱이 사용자를 인증할 때 사용되는 기본 인증 체계를 나타낸다.
                // 
                options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                // This indicates the authentication scheme that will be used by default when the app encounters an authentication challenge. 
                // Which authentication handler to use for responding to failed authentication or authorization attempts.
                // 
                options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
            })
            .AddJwtBearer(options =>
            {
                // Define token validation parameters to ensure tokens are valid and trustworthy
                options.TokenValidationParameters = new TokenValidationParameters
                {
                    ValidateIssuer = true, // Ensure the token was issued by a trusted issuer
                    ValidIssuer = builder.Configuration["Jwt:Issuer"], // The expected issuer value from configuration
                    ValidateAudience = false, // Disable audience validation (can be enabled as needed)
                    ValidateLifetime = true, // Ensure the token has not expired
                    ValidateIssuerSigningKey = true, // Ensure the token's signing key is valid
                    // Define a custom IssuerSigningKeyResolver to dynamically retrieve signing keys from the JWKS endpoint
                    IssuerSigningKeyResolver = (token, securityToken, kid, parameters) =>
                    {
                        //Console.WriteLine($"Received Token: {token}");
                        //Console.WriteLine($"Token Issuer: {securityToken.Issuer}");
                        //Console.WriteLine($"Key ID: {kid}");
                        //Console.WriteLine($"Validate Lifetime: {parameters.ValidateLifetime}");
                        // Initialize an HttpClient instance for fetching the JWKS
                        var httpClient = new HttpClient();
                        // Synchronously fetch the JWKS (JSON Web Key Set) from the specified URL
                        var jwks = httpClient.GetStringAsync($"{builder.Configuration["Jwt:Issuer"]}/.well-known/jwks.json").Result;
                        // Parse the fetched JWKS into a JsonWebKeySet object
                        var keys = new JsonWebKeySet(jwks);
                        // Return the collection of JsonWebKey objects for token validation
                        return keys.Keys;
                    }
                };
            });

            // 구성된 서비스와 미들웨어로 웹앱 인스턴스를 빌드
            var app = builder.Build();
            if (app.Environment.IsDevelopment()) { app.UseSwagger(); app.UseSwaggerUI(); } // 필요 x

            app.UseHttpsRedirection(); // 보안된 통신을 위해 HTTPS 리디렉션 적용
            app.UseAuthentication(); // Authentication 미들웨어 사용, 들어오는 JWT 토큰을 처리하고 유효성을 검사
            app.UseAuthorization(); // Authorization 미들웨어 사용, 사용자 역할 및 클레임에 따라 액세스 정책을 적용
            app.MapControllers();
            app.Run();
        }
    }
}
```



출처 : <br/>
https://dotnettutorials.net/lesson/jwt-authentication-in-asp-net-core-web-api/ <br/>
<hr/>


# 마무리 안됨
조금 더 확인해볼만한 것 <br/>
https://bigexecution.tistory.com/22 <br/>
https://jasonwatmore.com/net-6-jwt-authentication-with-refresh-tokens-tutorial-with-example-api#authorize-attribute-cs <br/>



<hr/>
