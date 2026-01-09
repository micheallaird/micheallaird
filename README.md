# Live Project

## Table of Contents
* [Snippets](#snippets)
* [Demonstration](#demonstation)
* [Screenshots](#screenshots)
* [Project Conclusion](#project-conclusion--lessons-learned)

## Introduction

In the last two weeks of my time in The Tech Academy Bootcamp, I worked on a MVC application with my fellow students using Azure Devops. Each of us were assigned stories that we could work on that added to the project as a whole. I was responsible for working on the full-stack development of the Blog area of the website. I had the opportunity to work on several full stack stories. To provide insight into my development process, I have documented key logic through code samples and supporting visual demonstrations. These snippets highlight the core architecture of the project, accompanied by screenshots that validate successful implementation and UI integration. 

<a id="snippets"></a>
## Snippets
### Index Snippet

<details>
<summary>Click to view the full Blog Author Index code</summary>

```cshtml
@model IEnumerable<TheatreCMS3.Areas.Blog.Models.BlogAuthor>

@{
    ViewBag.Title = "Author Index";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

@section Styles {
    <link rel="stylesheet" href="~/Content/Blog.css" />
}

<div class="container mt-4 CMS-author-details-container">
    <div class="d-flex justify-content-between align-items-center mb-4">
        <h2 class="cms-text-dark">Blog Authors</h2>
        <a href="@Url.Action("Create")" class="btn btn-success btn-sm d-inline-flex align-items-center">
            <i class="fa-solid fa-plus fa-fw"></i> <span>Create New</span>
        </a>
    </div>

    @* Render the Partial View for each author in the database *@
    <div class="CMS-author-list-wrapper">
        @foreach (var item in Model)
        {
            @Html.Partial("_BlogAuthor", item)
        }
    </div>
</div>

@section Scripts {
    <script>
        /**
         * NEW: Asynchronous Soft Delete Logic
         */
        function confirmSoftDelete(authorId) {
            if (confirm("Are you sure you want to mark this author as 'Left'?")) {
                const cardSelector = `#author-card-${authorId}`;
                const token = $(cardSelector).find('input[name="__RequestVerificationToken"]').val();

                $.ajax({
                    url: '@Url.Action("Delete", "BlogAuthors")',
                    type: 'POST',
                    data: {
                        id: authorId,
                        __RequestVerificationToken: token
                    },
                    success: function (response) {
                        if (response.success) {
                            $(cardSelector).fadeOut(500, function() {
                                $(this).remove();
                            });
                        } else {
                            alert("Error: " + response.message);
                        }
                    },
                    error: function () {
                        alert("An error occurred while trying to delete the author.");
                    }
                });
            }
        }

        /**
         * EXISTING: Tab Switching Logic
         */
        function showCardSection(section, authorId) {
            const sections = ['details', 'posts', 'contact', 'twitter'];

            sections.forEach(key => {
                const contentId = `${key}Sec-${authorId}`;
                const btnId = `btn${key.charAt(0).toUpperCase() + key.slice(1)}-${authorId}`;

                const contentElement = document.getElementById(contentId);
                const btnElement = document.getElementById(btnId);

                if (contentElement && btnElement) {
                    if (key === section) {
                        contentElement.style.display = 'block';
                        btnElement.classList.add('active');
                    } else {
                        contentElement.style.display = 'none';
                        btnElement.classList.remove('active');
                    }
                }
            });
        }
    </script>
}
```
</details>

### Create Snippet

<details>
<summary>Click to view the full Blog Author Create code</summary>

```cshtml
@model TheatreCMS3.Areas.Blog.Models.BlogAuthor

@{
    ViewBag.Title = "Create";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<h2>Create</h2>


@using (Html.BeginForm())
{
    @Html.AntiForgeryToken()
    <h4>BlogAuthor</h4>
    <div class="form-horizontal CMS-author-create-and-edit-form">
        <hr />
        @Html.ValidationSummary(true, "", new { @class = "text-danger" })
        <div class="form-group">
            @Html.LabelFor(model => model.Name, htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.Name, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.Name, "", new { @class = "text-danger" })
            </div>
        </div>

        <div class="form-group">
            @Html.LabelFor(model => model.Bio, htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @* Note: TextAreaFor attributes are passed directly as a new object *@
                @Html.TextAreaFor(model => model.Bio, new { @class = "form-control bio-editor", rows = "5" })
                @Html.ValidationMessageFor(model => model.Bio, "", new { @class = "text-danger" })
            </div>
        </div>


        <div class="form-group">

            <div class="col-md-10">
                <!-- We use 'd-flex justify-content-center' to center the sub-fields -->
                <!-- 'gap-3' (Bootstrap 5) or manual margins adds space between the two small fields -->
                <div class="row d-flex justify-content-center">

                    <div class="col-md-5">
                        <!-- Smaller column size (5 instead of 6 or 10) -->
                        @Html.LabelFor(model => model.Joined, htmlAttributes: new { @class = "small" })
                        @Html.EditorFor(model => model.Joined, new { htmlAttributes = new { @class = "form-control", type = "datetime-local" } })
                        @Html.ValidationMessageFor(model => model.Joined, "", new { @class = "text-danger" })
                    </div>

                    <div class="col-md-5">
                        <!-- Same smaller column size -->
                        @Html.LabelFor(model => model.Left, htmlAttributes: new { @class = "small" })
                        @Html.EditorFor(model => model.Left, new { htmlAttributes = new { @class = "form-control", type = "datetime-local" } })
                        @Html.ValidationMessageFor(model => model.Left, "", new { @class = "text-danger" })
                    </div>

                </div>
            </div>
        </div>



        <div class="form-group button-style">

            <!-- ActionLink is perfect for navigation (Back to List) -->
            @Html.ActionLink("Back to List", "Index", null, new { @class = "btn btn-secondary px-4" })

            <!-- Submit button to save data -->
            <input type="submit" value="Create" class="btn btn-success px-4" />
        </div>

    </div> <!-- End of CMS-author-create-and-edit-form -->
}
@section Scripts {
    @Scripts.Render("~/bundles/jqueryval")
}
```
</details>

### Edit Snippet

<details>
<summary>Click to view the full Blog Author Edit code</summary>

```cshtml
@model TheatreCMS3.Areas.Blog.Models.BlogAuthor

@{
    ViewBag.Title = "Edit";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<h2>Edit</h2>

@using (Html.BeginForm())
{
    @Html.AntiForgeryToken()
    <h4>BlogAuthor</h4>
    @* Added the same wrapper class as the Create page *@
<div class="form-horizontal CMS-author-create-and-edit-form">
    <hr />
    @Html.ValidationSummary(true, "", new { @class = "text-danger" })
    @Html.HiddenFor(model => model.BlogAuthorId)

    <div class="form-group">
        @Html.LabelFor(model => model.Name, htmlAttributes: new { @class = "control-label col-md-2" })
        <div class="col-md-10">
            @Html.EditorFor(model => model.Name, new { htmlAttributes = new { @class = "form-control" } })
            @Html.ValidationMessageFor(model => model.Name, "", new { @class = "text-danger" })
        </div>
    </div>

    <div class="form-group">
        @Html.LabelFor(model => model.Bio, htmlAttributes: new { @class = "control-label col-md-2" })
        <div class="col-md-10">
            @* Note: TextAreaFor attributes are passed directly as a new object *@
            @Html.TextAreaFor(model => model.Bio, new { @class = "form-control bio-editor", rows = "5" })
            @Html.ValidationMessageFor(model => model.Bio, "", new { @class = "text-danger" })
        </div>
    </div>


    @* Date fields restructured to match the Create page layout *@
    <div class="form-group">
        <div class="col-md-10">
            <div class="row d-flex justify-content-center">
                <div class="col-md-5">
                    @Html.LabelFor(model => model.Joined, htmlAttributes: new { @class = "small" })
                    @Html.EditorFor(model => model.Joined, new { htmlAttributes = new { @class = "form-control", type = "datetime-local" } })
                    @Html.ValidationMessageFor(model => model.Joined, "", new { @class = "text-danger" })
                </div>

                <div class="col-md-5">
                    @Html.LabelFor(model => model.Left, htmlAttributes: new { @class = "small" })
                    @Html.EditorFor(model => model.Left, new { htmlAttributes = new { @class = "form-control", type = "datetime-local" } })
                    @Html.ValidationMessageFor(model => model.Left, "", new { @class = "text-danger" })
                </div>
            </div>
        </div>
    </div>

    @* Replaced standard div with the button-style container *@
    <div class="form-group button-style">
        @Html.ActionLink("Back to List", "Index", null, new { @class = "btn btn-secondary px-4" })
        <input type="submit" value="Save" class="btn btn-success px-4" />
    </div>

</div> <!-- End of CMS-author-create-and-edit-form -->
}

@section Scripts {
    @Scripts.Render("~/bundles/jqueryval")
}
```
</details>

### Controller Snippet

<details>
<summary>Click to view the full Blog Author Controller code</summary>

```cshtml

using System;
using System.Collections.Generic;
using System.Data;
using System.Data.Entity;
using System.Linq;
using System.Net;
using System.Web;
using System.Web.Mvc;
using TheatreCMS3.Areas.Blog.Models;
using TheatreCMS3.Models;

namespace TheatreCMS3.Areas.Blog.Controllers
{
    public class BlogAuthorsController : Controller
    {
        private ApplicationDbContext db = new ApplicationDbContext();

        // GET: Blog/BlogAuthors
        public ActionResult Index()
        {
            return View(db.BlogAuthors.ToList());
        }

        // GET: Blog/BlogAuthors/Details/5
        public ActionResult Details(int? id)
        {
            if (id == null)
            {
                return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
            }
            BlogAuthor blogAuthor = db.BlogAuthors.Find(id);
            if (blogAuthor == null)
            {
                return HttpNotFound();
            }
            // Swapped the delete page for details page to remove redundancy (The Delete Button now returns only the Details Page)
            return View("Delete", blogAuthor);
        }

        // GET: Blog/BlogAuthors/Create
        public ActionResult Create()
        {
            return View();
        }

        // POST: Blog/BlogAuthors/Create
        // To protect from overposting attacks, enable the specific properties you want to bind to, for 
        // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create([Bind(Include = "BlogAuthorId,Name,Bio,Joined,Left")] BlogAuthor blogAuthor)
        {
            if (ModelState.IsValid)
            {
                db.BlogAuthors.Add(blogAuthor);
                db.SaveChanges();
                return RedirectToAction("Index");
            }

            return View(blogAuthor);
        }

        // GET: Blog/BlogAuthors/Edit/5
        public ActionResult Edit(int? id)
        {
            if (id == null)
            {
                return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
            }
            BlogAuthor blogAuthor = db.BlogAuthors.Find(id);
            if (blogAuthor == null)
            {
                return HttpNotFound();
            }
            return View(blogAuthor);
        }

        // POST: Blog/BlogAuthors/Edit/5
        // To protect from overposting attacks, enable the specific properties you want to bind to, for 
        // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Edit([Bind(Include = "BlogAuthorId,Name,Bio,Joined,Left")] BlogAuthor blogAuthor)
        {
            if (ModelState.IsValid)
            {
                db.Entry(blogAuthor).State = EntityState.Modified;
                db.SaveChanges();
                return RedirectToAction("Index");
            }
            return View(blogAuthor);
        }

        // GET: Blog/BlogAuthors/Delete/5
        public ActionResult Delete(int? id)
        {
            if (id == null)
            {
                return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
            }

            // REMOVED .Include(a => a.BlogPosts) because the property was deleted from the Model
            BlogAuthor blogAuthor = db.BlogAuthors.Find(id);

            if (blogAuthor == null)
            {
                return HttpNotFound();
            }
            // Swapped the delete page for details page to remove redundancy with links
            return View("Details", blogAuthor);
        }


        // POST: Blog/BlogAuthors/Delete/5
        [HttpPost, ActionName("Delete")]
        [ValidateAntiForgeryToken]
        public ActionResult DeleteConfirmed(int id)
        {
            var blogAuthor = db.BlogAuthors.Find(id);
            if (blogAuthor != null)
            {
                // 2026 Soft Delete Logic: Set 'Left' date instead of removing record
                blogAuthor.Left = DateTime.Now;
                db.Entry(blogAuthor).State = EntityState.Modified;
                db.SaveChanges();

                return Json(new { success = true });
            }
            return Json(new { success = false, message = "Author record not found." });
        }


        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                db.Dispose();
            }
            base.Dispose(disposing);
        }
    }
}
```
</details>

### Partial View Snippet

<details>
<summary>Click to view the full Blog Author Partial View code</summary>

```cshtml
@model TheatreCMS3.Areas.Blog.Models.BlogAuthor

<div id="author-card-@Model.BlogAuthorId" class="CMS-author-details-container mb-4 p-0">
    @* TAB NAVIGATION (Top of Card) *@
    <div class="CMS-tab-centering-container">
        <div class="CMS-tab-wrapper">
            <button type="button" class="CMS-tab-btn active" id="btnDetails-@Model.BlogAuthorId" onclick="showCardSection('details', '@Model.BlogAuthorId')">Author Details</button>
            <button type="button" class="CMS-tab-btn" id="btnPosts-@Model.BlogAuthorId" onclick="showCardSection('posts', '@Model.BlogAuthorId')">Blog Posts</button>
            <button type="button" class="CMS-tab-btn" id="btnContact-@Model.BlogAuthorId" onclick="showCardSection('contact', '@Model.BlogAuthorId')">Contact</button>
            <button type="button" class="CMS-tab-btn" id="btnTwitter-@Model.BlogAuthorId" onclick="showCardSection('twitter', '@Model.BlogAuthorId')">Twitter</button>
        </div>
    </div>

    @* MAIN CONTENT CARD *@
    <div class="CMS-author-grid shadow-sm">
        <div class="CMS-author-image-wrapper">
            <img src="~/Content/Images/placeholder.jpg" class="CMS-author-img" alt="Author Image">
        </div>

        <div class="CMS-author-content-wrapper d-flex flex-column">
            <div class="flex-grow-1">
                @* Section: Details *@
                <div id="detailsSec-@Model.BlogAuthorId">
                    <strong class="cms-text-dark CMS-author-bio-name">@Model.Name</strong>
                    <p class="text-dark">@Html.DisplayFor(model => model.Bio)</p>

                    @* ALIGNED ROW: Dates and Actions on the same line *@
                <div class="d-flex justify-content-between align-items-end mt-3 text-dark CMS-author-dates">
                    @* Dates Side *@
                    <div class="d-flex">
                        <div class="mr-4 me-4">
                            <strong class="cms-text-dark d-block">@Html.DisplayNameFor(model => model.Joined)</strong>
                            <span>@Model.Joined.ToString("g")</span>
                        </div>
                        @if (Model.Left.HasValue)
                        {
                            <div>
                                <strong class="cms-text-dark d-block">@Html.DisplayNameFor(model => model.Left)</strong>
                                <span>@Model.Left.Value.ToString("g")</span>
                            </div>
                        }
                    </div>
                    @* Action Buttons Side *@
                    <div class="CMS-author-actions d-flex align-items-stretch">
                        @Html.AntiForgeryToken()

                        @* Use btn-sm-fixed custom class to ensure consistency *@
                        <a href="@Url.Action("Edit", new { id = Model.BlogAuthorId })" class="btn btn-success btn-sm me-1 mr-1 d-flex align-items-center justify-content-center">
                            <i class="fa-solid fa-pencil"></i> <span class="ms-1 d-none d-md-inline">Edit</span>
                        </a>

                        <button type="button" class="btn btn-danger btn-sm d-flex align-items-center justify-content-center" onclick="confirmSoftDelete(@Model.BlogAuthorId)">
                            <i class="fa-solid fa-trash-can fa-fw"></i>
                        </button>
                    </div>



                </div>
                </div>

                @* Placeholder Sections for other tabs (Hidden) *@
                <div id="postsSec-@Model.BlogAuthorId" style="display: none;"><p class="text-dark italic">Blog posts for @Model.Name.</p></div>
                <div id="contactSec-@Model.BlogAuthorId" style="display: none;"><p class="text-dark italic">Contact @Model.Name.</p></div>
                <div id="twitterSec-@Model.BlogAuthorId" style="display: none;"><p class="text-dark italic">Twitter feed for @Model.Name.</p></div>
            </div>
        </div>
    </div>
</div>
```
</details>

### CSS Snippet

<details>
<summary>Click to view the full Blog Author CSS code</summary>

```cshtml

/* ▼ This ▼ is for you to easily reference the colors on the Site.css page */
/* Do not uncomment these color variables */

/* CSS color variables */
/*:root { /* Color palette for css */
/* --main-color: #BD1A11;  Red */
/*--main-color--light: #F04D44;*/ /* Red, a shade lighter */
/*--secondary-color: #D6972A;*/ /* Yellow gold */
/*--light-color: #FFFBFB;*/ /* White */
/*--dark-color: #000000;*/ /* Black */
/*--secondary-color--dark: #9D7C39;*/ /* Dark Gold */
/*}*/
/* Blog.css */

:root {
    --light-color: #FFFBFB;
    --dark-frame: #000000;
    --tab-inactive: #4a4a4a;
    --success-green: #28a745; /* Added missing variable */
    --border-color: #dee2e6;
    --secondary-color--dark: #9D7C39;
}

/*Styling for the Create page in the Blog Authors section*/

.CMS-author-create-and-edit-form {
    background-color: var(--main-color--light);
    border-radius: 10px;
}

/* Adds padding to the top and the bottom of the buttons*/
.button-style {
    padding: 35px 15px 20px 15px;
}


/* --- DETAILS PAGE: INTEGRATED TAB NAVIGATION --- */

/* Centers the entire tab group */
.CMS-tab-centering-container {
    display: flex;
    justify-content: center;
    width: 100%;
}

/* The light-colored container that 'wraps' up from the bio section */
.CMS-tab-wrapper {
    display: inline-flex;
    background-color: var(--light-color);
    padding: 5px 3px 0 3px;
    border-radius: 10px 10px 0 0;
}

/* Base Button Style */
div.CMS-tab-wrapper button.CMS-tab-btn {
    background-color: var(--tab-inactive);
    color: white;
    border: none;
    padding: 10px 25px;
    font-weight: bold;
    cursor: pointer;
    border-radius: 8px;
    transition: background-color 0.2s ease;
    margin: 0 4px;
}

    div.CMS-tab-wrapper button.CMS-tab-btn:hover:not(.active) {
        background-color: #5a5a5a;
    }

    /* THE GREEN ACTIVE BUTTON: High specificity ensures this stays green */
    div.CMS-tab-wrapper button.CMS-tab-btn.active {
        background-color: var(--success-green);
        color: #ffffff;
        border: 1px solid rgba(0,0,0,0.1);
        box-shadow: 0 2px 4px rgba(0,0,0,0.2);
    }

/* --- MAIN CONTENT CARD --- */

.CMS-author-details-container {
    background-color: var(--dark-frame);
    padding: 2rem;
    border-radius: 10px;
}

.CMS-author-grid {
    display: flex;
    background-color: var(--light-color);
    /* Set borders individually to omit the top one */
    border-left: 1px solid var(--border-color);
    border-right: 1px solid var(--border-color);
    border-bottom: 1px solid var(--border-color);
    border-top: none;
    border-radius: 5px;
    overflow: hidden;
    position: relative;
    z-index: 1;
    margin-top: -1px;
    min-height: 300px;
}

/* Author Bio Name */
div.CMS-author-grid div.CMS-author-content-wrapper strong.CMS-author-bio-name {
    font-size: 1.5rem;
    display: inline-block;
    margin-bottom: 0.5rem;
}

/* Social Media Icons Styling */
.social-icon {
    color: var(--dark-frame);
    font-size: 1.2rem;
    margin-left: 8px;
    transition: color 0.3s ease;
}

    .social-icon:hover {
        color: var(--success-green); /* Changes color on hover */
        text-decoration: none;
    }

/* Ensure the container clears floated elements */
.CMS-author-details-container::after {
    content: "";
    display: block;
    clear: both;
}


/* Sidebar Image */
.CMS-author-image-wrapper {
    flex: 0 0 33.33%;
    border-right: none;
}

.CMS-author-img {
    width: 100%;
    height: 100%;
    object-fit: contain;
    display: block;
}

/* Content Area */
.CMS-author-content-wrapper {
    flex: 1;
    padding: 2rem;
}

/* --- TYPOGRAPHY & SPACING --- */

.cms-text-secondary-dark {
    color: var(--secondary-color--dark);
}

.CMS-author-content-wrapper strong.CMS-author-bio-name {
    font-size: 1.8rem;
    display: inline-block;
    margin-bottom: 0.5rem;
}

.CMS-author-dates strong {
    display: block;
    margin-bottom: 2px;
}

/* --- FOOTER BUTTONS --- */

.CMS-footer-wrapper {
    margin-top: .5rem;
    display: flex;
    align-items: stretch; /* Keeps all buttons at equal height */
}

    .CMS-footer-wrapper .btn {
        margin-right: 8px;
        display: inline-flex;
        align-items: center;
        padding: .5rem 1.25rem; /* Standard padding for links */
        height: 100%;
    }

    .CMS-footer-wrapper form {
        margin-left: auto;
        margin-right: 0;
        display: flex;
    }

        /* Specific styling for the Delete button to make it smaller in width */
        .CMS-footer-wrapper form .btn-danger {
            margin-right: 0;
            height: 100%;
            align-self: stretch;
            padding: 1rem 0.75rem; /* Reduced horizontal padding to narrow the button */
            justify-content: center; /* Centers the icon/text in the narrower space */
        }

    .CMS-footer-wrapper .btn i {
        margin-right: 5px;
    }


/* --- MOBILE RESPONSIVENESS --- */

@media (max-width: 768px) {
    .CMS-author-grid {
        flex-direction: column;
    }

    .CMS-author-image-wrapper {
        flex: 0 0 auto;
        height: 300px;
        border-right: none;
        border-bottom: none;
    }

    .CMS-author-details-container {
        padding: 1rem;
    }

    div.CMS-tab-wrapper button.CMS-tab-btn {
        padding: 10px 15px;
        font-size: 0.9rem;
    }
}
```
</details>

## Demonstation

![LP Animation](https://github.com/user-attachments/assets/6f53b87a-fb52-4667-8961-3a33cf33ca47)

## Screenshots

<img width="1325" height="898" alt="Screenshot 2026-01-08 122246" src="https://github.com/user-attachments/assets/1281a9a9-7816-49f8-a82d-6aa62c085c9b" />
<img width="1325" height="903" alt="Screenshot 2026-01-08 122310" src="https://github.com/user-attachments/assets/7570dfc4-307f-4c37-bf68-5d2623a25835" />
<img width="1328" height="891" alt="Screenshot 2026-01-08 122356" src="https://github.com/user-attachments/assets/d5b0c37d-8f48-4ce7-9055-24518e709a25" />
<img width="1303" height="946" alt="Screenshot 2026-01-08 122419" src="https://github.com/user-attachments/assets/cf414cc9-5697-4139-a51a-7a47eb975728" />

## Project Conclusion & Lessons Learned

This Live Project was the culmination of my time at The Tech Academy, allowing me to apply the skills I've acquired in a real-world simulation. It provided hands-on experience and a deeper understanding of the full software development lifecycle.

### Key Accomplishments

* Applied Full-Stack Knowledge: I successfully integrated front-end (HTML, CSS, JavaScript) and back-end (e.g., C# and SQL) technologies to add on to a functional application.
* Version Control Proficiency: I utilized Git and GitHub for version control, managing code changes and collaborating effectively.
* Problem-Solving & Debugging: I developed debugging skills by systematically identifying and resolving issues encountered during they project sprints.
* Project Management Fundamentals: I applied Agile and Scrum methodologies to organize my workflow, set goals, and manage the project efficiently.
  
### Lessons Learned
Through this project, I gained several key insights:

* The Value of Planning: Thorough initial planning and design, including considering edge cases and potential failure modes, worked through the development process.
* Documentation: Clear and concise documentation (both in-code comments and a README) is essential for maintainability and for other developers to understand the project.
* Iterative Development: Breaking down complex problems into smaller, manageable tasks allowed me to make progress and avoid getting stuck striving for initial perfection.
* Importance of Testing: Rigorous testing throughout the development cycle ensures a more stable and reliable product.

[Back to Top](#live-project)
