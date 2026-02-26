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
5. Make ur Controller AccountService
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
                IdentityResult res = await UserManager.CreateAsync(user);
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

### <h3 style="color: tomato;"> Remember That</h3>
| Table | Class | Service | Repository| Context|
|-----------|---------|------------|------------|------------|
|User | IdentityUser | UserManager | UserStore | IdentityDbContext 
|Role | IdentityRole | RoleManager | RoleStore | IdentityDbContext 

