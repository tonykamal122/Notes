# Identity `Notes`

## Steps to make identity Layer

1. Install Package Identity `Microsoft.AspNetCore.Identity.EntityFrameworkCore`
   - If you use <span style = "color: #2d8ff1ff">3 Tier Arch</span> (Install it at DAL)

2. Create Class ApplicationUser that Inhert from the non generic class IdentityUser
   - Why non generic class ? `For The Guid id`
   - Why We Inheret From it ? to add columns at the IdentityUser because its a read only Class

   ```csharp
   public class ApplicationUser : IdentityUser{
    public string? Address {get; set;}
   }
   ```

3. ```csharp
   public class OurContext : IdentityDbContext<ApplicationUser>{
   }
   ```
   - Why we make that? to do not make 2 databases so ourcontext will get the db context from the identitydbcontext and to get the identitydbcontext dbsets 
   - Remember to add ` base.OnModelCreating(modelBuilder);` at `OnModelCreating(...)`

4. Add - Migration
5. Make ur Controller AccountService , ViewModel , Mapping (viewmodel -> ApplicationUser
) , Create User Using `UserManager` then check if it created using IdentityResult
we make the cookie using `SignInManager`
   ```csharp
   public class AccountController : Controller
    {
        private readonly UserManager<ApplicationUser> UserManager;

        public AccountController(UserManager<ApplicationUser> UserManager, SignInManager<ApplicationUser> signInManager)
        {
            this.UserManager = UserManager;
            SignInManager = signInManager;
        }

        public SignInManager<ApplicationUser> SignInManager { get; }

        [HttpGet]
        public IActionResult Register(UserViewModel userViewModel)
        {
            return View("Login", userViewModel);
        }
        [HttpPost]
        public async Task<IActionResult> SaveRegister(UserViewModel userViewModel)
        {
            if (ModelState.IsValid)
            {
                //awel haga map from userViewModel to identityuser aw (applicationuser ya3ni) tab ana 3ayz
                //service b2a el maska el repository esmaha usermanager haro7 a3malha inject bas awel haga ana m7tag
                //a3mel applicationuser el m7tago el manager
                ApplicationUser user = new ApplicationUser()
                {
                    UserName = userViewModel.UserName,
                    Email = userViewModel.Email,
                    PasswordHash = userViewModel.Password,
                };
                //create user b2a bel applicationuser
                IdentityResult res = await UserManager.CreateAsync(user,userViewModel.Password);
                //create user cookie b2a ezay ? bel signin manager
                if (res.Succeeded)
                {
                    //cookie
                    //El SignInManager Ha3malo Inject Bardo msh ha3raf a3malo
                    await SignInManager.SignInAsync(user, false);
                    //false yab2a session dah el tabe3y fel register 3shan ya3mel 
                    //login tany la2an law m3malsh ba3d kda haynsa el password aslan
                    return RedirectToAction("Index", "Home");
                }
                foreach (var it in res.Errors)
                {
                    ModelState.AddModelError("", it.Description);
                }
            }
            return View("Register", userViewModel);
        }
   ```
6. Register Service in the `Program.cs`
   ```csharp
   builder.Services.AddIdentity<ApplicationUser, IdentityRole>()
   .AddEntityFrameworkStores<OurContext>()
   .AddDefaultTokenProviders();

   app.UseAuthentication();
   app.UseAuthorization();
   [Authentication] => to doesn't make it anonymous
   ```
   - We just need to register the identity 
   - We Iject `AddIdentity` msh `AddIdentityCore` leah b2a ? 3shan
   IdentityCore bta3mel el user bas 
7.  Login View Model and but validation on it 
### <h3 style="color: tomato;"> Remember That</h3>
| Table | Class | Service | Repository| Context|
|-----------|---------|------------|------------|------------|
|User | IdentityUser | UserManager | UserStore | IdentityDbContext 
|Role | IdentityRole | RoleManager | RoleStore | IdentityDbContext 

### Note `CreateAsync` have 2 overloads with password and ignore password 
- await UserManager.CreateAsync(user,userViewModel.Password);
  - so we can use it to create user with password because it will hash it
    and put it in the application user but remembar that have specific constrains
    for the password so you can change it from the `Program.cs (Servce Register)` like this:
    ```csharp
    builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options =>
    {
        options.Password.RequireDigit = true;
        options.Password.RequireLowercase = true;
        options.Password.RequireUppercase = true;
        options.Password.RequireNonAlphanumeric = true;
        options.Password.RequiredLength = 8;
    })
    .AddEntityFrameworkStores<OurContext>()
    .AddDefaultTokenProviders();
    ```
- await UserManager.CreateAsync(user);
  - so we can use it to create user without password
- Note `ModelState.AddModelError("", it.Description);`
  - so we can use it to add error to the model state
- For Client Side Validation we should make vm and at the View
  ```
    @section Scripts{
        <partial name="_ValidationScriptsPartial" />
    }```
- and at the layout we should put
 ```
 @RenderSection("Scripts",false)
 ```
## OBSRIVATION
- leah b2a fel `signInManager.SignInAsync(user,false)` leah bykoun 3ayez el user?
asl byakhoud byanat men el user dah we y7otaha fel cookie 3shan lma tegy tany mayro7sh
yagebha men el data base ygebha men el cookie fa tab2a asra3 we a2rab

`Eh el bayant el bbkhazan fel cookie [Claims]`:
  - Id
  - Name
  - Role if he have one

khaly balak b2a 3shan te2ra el cookie deh msh hta3mel
`context.httpContext.cookie` leh b2a? 3shan msh enta el 3amelha aslan

## `Tmam how to access cookie b2a ?`
- 3an tare2 el controller byaras kza haga menha eh ?
- viewdata , viewbag and (`UserPrinciple User`)that see the request
and if find the cookie identity it take the data from it and store it at a
collection called `ClaimsPrincipal` => [Key : Value]  serializa in cookie

- law msh [authroied] hay2oulk isauthanticated = false we kman el claimsprincipal = empty


`User.Identity.IsAuthenticated` => to check if the user is authenticated or not
so we can make diffrent behaviors

## Note
tab law ana 3ayez el id b2a? 
`Claim idClaim = User.Claims.firstorDefault(c => c.Type == ClaimTypes.NameIdentifier)`
User is a context then make a query like
context.Claims.FirstOrDefault(c => c.Type == ClaimTypes.NameIdentifier)
var id = idClaim?.Value;
