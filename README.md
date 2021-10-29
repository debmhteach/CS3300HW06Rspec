# Iteration 04: Testing Rails with RSpec

Beginners introduction to testing Ruby On Rails application with RSpec and Capybara.


## Install RSpec with Rails

1. Add `rspec` and `mini_racer` to the `Gemfile`:

```ruby
  gem 'mini_racer'
  
  group :development, :test do
    gem 'rspec-rails', '~> 3.7'
  end

```

2. Install dependencies: 
```ruby
bundle install
```

3. Execute the following command to create a `/spec` dir;
```ruby 
bundle exec rails generate rspec:install
``` 

4. Run rspec: 
```ruby 
bundle exec rspec # Make sure you're in the root directory of the application when you run this command
```

## Implement Tests
A. Your first spec 


1. Create first spec `spec/example/first_spec.rb`:

```ruby

require "rails_helper"

RSpec.describe "The math below is wrong..." do
  it "should equal 42 but we said 43" do
    expect(6 * 7).to eq(43)
  end
end

```

2. Run rspec ./spec/example/first_spec.rb
3. And catch an error:

```

Failures:

  1) hello spec math should be able to perform basic math
     Failure/Error: expect(6 * 7).to eq(43)
     
       expected: 43
            got: 42


```

3. Fix the error to get test passed:

```ruby

require "rails_helper"

RSpec.describe "The math below is right..." do
  it "should equal 42" do
    expect(6 * 7).to eq(42)
  end
end

```

4. Add another spec with empty string:

```ruby

require "rails_helper"

RSpec.describe "hello spec" do
  # ...
  describe String do
    let(:string) { String.new }
    it "should provide an empty string" do
      expect(string).to eq("")
    end
  end
 end
```

B. Create a unit test for Project model

1. Create `spec/models/project_spec.rb`:

```ruby

require "rails_helper"

RSpec.describe Project, type: :model do
  context "validations tests" do
    it "ensures the title is present" do
      project = Project.new(description: "Content of the description")
      expect(project.valid?).to eq(false)
    end

    
    it "should be able to save project" do
      project = Project.new(title: "Title", description: "Some description content goes here")
      expect(project.save).to eq(true)
    end
  end

  context "scopes tests" do

  end
end

```



2. The test should fail because the project is not validating that a title is entered. 
Update your model project.rb to require a title is entered using validates_presence_of. 

Re-run the RSpec tests and the required title should pass.

3. Update your project_spec.rb code to test that the project has a description. Run test. Then update your project.rb model code to require a description to be entered.


4. Add scope specs to the spec/models/project_spec.rb`

```ruby

require "rails_helper"

RSpec.describe Project, type: :model do
  # ...

  context "scopes tests" do
    let(:params) { { title: "Title", description: "some description" } }
    before(:each) do
      Project.create(params)
      Project.create(params)
      Project.create(params)
    end

    it "should return all projects" do
      expect(Project.count).to eq(3)
    end

  end
end

```


C.  Create functional test for Projects controller

1. Create Projects spec `spec/controller/projects_spec.rb`:

```ruby

require "rails_helper"

RSpec.describe ProjectsController, type: :controller do
  context "GET #index" do
    it "returns a success response" do
      get :index
      # expect(response.success).to eq(true)
      expect(response).to be_success
    end
  end

  context "GET #show" do
    let!(:project) { Project.create(title: "Test title", description: "Test description") }
    it "returns a success response" do
      get :show, params: { id: project }
      expect(response).to be_success
    end
  end
end

```

D. Create integration spec: home page 


1. Create first feature test by running `bundle exec rails g rspec:feature home_page`:

```ruby

require "rails_helper"

RSpec.feature "Visiting the homepage", type: :feature do
  scenario "The visitor should see projects" do
    visit root_path
    expect(page).to have_text("Projects")
  end
end

```


D. Create integration spec: Projects


1. `bundle exec rails g rspec:feature projects`

2. Full Projects test:

```ruby

require 'rails_helper'

RSpec.feature "Projects", type: :feature do
  context "Create new project" do
    before(:each) do
      visit new_project_path
      within("form") do
        fill_in "Title", with: "Test title"
      end
    end

    scenario "should be successful" do
      fill_in "Description", with: "Test description"
      click_button "Create Project"
      expect(page).to have_content("Project was successfully created")
    end

    scenario "should fail" do
      click_button "Create Project"
      expect(page).to have_content("Description can't be blank")
    end
  end

  context "Update project" do
    let(:project) { Project.create(title: "Test title", description: "Test content") }
    before(:each) do
      visit edit_project_path(project)
    end

    scenario "should be successful" do
      within("form") do
        fill_in "Description", with: "New description content"
      end
      click_button "Update Project"
      expect(page).to have_content("Project was successfully updated")
    end

    scenario "should fail" do
      within("form") do
        fill_in "Description", with: ""
      end
      click_button "Update Project"
      expect(page).to have_content("Description can't be blank")
    end
  end

  context "Remove existing project" do
    let!(:project) { Project.create(title: "Test title", description: "Test content") }
    scenario "remove project" do
      visit projects_path
      click_link "Destroy"
      expect(page).to have_content("Project was successfully destroyed")
      expect(Project.count).to eq(0)
    end
  end
end

```


## Add Test Coverage 
A. Add simplecov gem:


1. Rails coverage report: `bundle exec rails stats`

2. To install simplecov add to your `Gemfile` to the `:test group`:

```ruby

gem 'simplecov', require: false

```

3. Add to your `spec/rails_helper.rb`:

```ruby

require 'simplecov'
SimpleCov.start 'rails' do
  add_filter '/bin/'
  add_filter '/db/'
  add_filter '/spec/' # for rspec
end

```


4. bundle install

5.  Running `bunlde exec rspec` will generate a coverage report 


## Resources
* https://relishapp.com/rspec/rspec-rails/docs 

* https://github.com/teamcapybara/capybara 

* https://github.com/colszowka/simplecov 

* http://www.betterspecs.org/  



