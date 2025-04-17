
# JWT

컨트롤러 <br/>
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

            // Retrieve the user from the Users table by matching the email (case-insensitive)
            // Also include the UserRoles and associated Roles for later use
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
            // Retrieve the active signing key from the SigningKeys table
            var signingKey = _context.SigningKeys.FirstOrDefault(k => k.IsActive);
            if (signingKey == null) { throw new Exception("No active signing key available."); }

            // Base64로 인코딩된 개인키 문자열을 바이트 배열로 다시 변환한다.
            var privateKeyBytes = Convert.FromBase64String(signingKey.PrivateKey);
            // 암호화 작업을 위한 새 RSA 인스턴스 생성
            // RSA 개인키를 RSA 인스턴스로 가져옴
            // RSA 인스턴스를 사용하여 새 RsaSecurityKey 생성
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



출처 : <br/>
https://dotnettutorials.net/lesson/jwt-authentication-in-asp-net-core-web-api/ <br/>
<hr/>
