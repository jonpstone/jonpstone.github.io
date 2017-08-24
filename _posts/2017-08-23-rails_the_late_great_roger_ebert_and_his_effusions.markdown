---
layout: post
title:  "Rails: The Late, Great Roger Ebert and his Effusions"
date:   2017-08-23 12:57:08 -0400
---


![](http://static.rogerebert.com/uploads/blog_post/primary_image/balder-and-dash/still-present-memories-of-roger-ebert-a-year-after-his-passing/primary_roger_ebert_54396.jpg)

**Background**

It's finally here. Two restarts, wrestling with Devise and then abondoning it when it wouldn't play ball with Omniauth, Carrierwave not on it's best behavoir, you name it. What I settled on for an idea was a web app that publishes scathing reviews of (what are generally considered to be by critics) terrible movies. The idea is that the app would be a collection point for the most, and quite often humorous reviews of bad movies. Needless to say, Roger Ebert (who until his death in 2013, was widley considered the Dean of American film critics), features heavily as he was famous for his reviews being so inventively mean that they were considered comedic. There would be no particular category of any one type of movie in this app, indeed, `Genre` was one of my central models. I wrote the code as though the app was to be created for a client. So I tried my best to abstract away any need whatsoever for what I considered to be a need for the client to go into (or re-hire me to go back into) the source code.

**Models and Migrations**

Of course, one of the many wonderful things (in this case a saving grace), about Rails, is that getting a function app off the ground and running can be done so rapidly. So, when you want to scrap everything and start over, twice, it really helps. I decided on three fundamental models that would form the core of the app: `Writer`, `Review` and `Genre`; the latter as stated above. `Genre` would actually be a very simple model that would only contain one attribute `genre_name`. The `Writer`, was effectively a kind of pseudo `User` model. It was seperate from the actual `User` model that I'll get to further down. I mean this in the sense that you do not **sign in** as a `Writer` - *this is not a blog app, this is not a blog app, this is not a blog app.* The `Writer` has very similar attributes to what a `User` would have on say, a social media platform, there's a `username` attribute, a `bio`, who they work for, etc. I saw no need to create an Active Record association between these two particular models. I did, as required, include a `has_many :through` association between `Genre` and `Review`, which included the attendent join table, as well as a `has_many` `belongs_to` relationship between `Writer` and `Review`. The `Review` model was probably the most complex so I'll just throw it out in a code snippet.

**Fig.1**
*./app/models/review.rb*

```
class Review < ApplicationRecord
  belongs_to :writer, inverse_of: :reviews
  has_many :review_genres
  has_many :genres, through: :review_genres
  has_many :comments, as: :commentable

  validates_presence_of :title, :content, :year, :date_published
  validates_uniqueness_of :title, :content
  validates :content, length: { minimum: 500 }
  validates :title, length: { minimum: 2 }
  validates :year, length: { is: 4 }

  mount_uploader :image, ImageUploader
  mount_uploader :banner, BannerUploader

  def self.latest_review
    order("created_at DESC").limit(1).first
  end

  def self.second_latest_review
    order("created_at DESC").offset(1).limit(1).first
  end

  def self.third_latest_review
    order("created_at DESC").offset(2).limit(1).first
  end

  def self.fourth_latest_review
    order("created_at DESC").offset(3).limit(1).first
  end
	
	  def self.last_five_reviews
    last(5).reverse
  end

  def self.ordered
    order(:title).all
  end
end
```

I won't go into the specific validations for the models, most of them are pretty arbitrary or are based on common sense approaches to protect the app from bad data. Note the uploaders and the  `has_many` association with `Comment`. I decided to follow a thrid party tutorial for this and it was my first run in with polymorphic associations. You can check out the tutorial I followed [here](https://www.codementor.io/ruby-on-rails/tutorial/threaded-comments-polymorphic-associations). Below is the `Comment` model, which is not as intimidating as one may think.

**Fig.2**
*./app/models/comment.rb*

```
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
  has_many :comments, as: :commentable

  validates :body, length: { minimum: 3 }
end
```

Aside from there being a `Review_genre` join model for the `Review` and `Genre` association, I'll quickly cover more of the `User` model here, in particular, the `omniauth` gem. So, getting this working was quite a process and I basically watched Avi's lecture on it right the end to eventually be able to get it right. However, after all, it works a treat and I think I have a decent understanding of how the double handshake works and how the third party app authenticates a `User`. Below is the model including the class method required to write to the model. As an aside, after a fair bit of frustration, I decided to forgo Devise Pundit/CanCan and rolled my own authentication and authorization in this app.

**Fig.3**
*./app/models/user.rb*

```
class User < ApplicationRecord
  has_secure_password
  validates_presence_of :username, :email
  validates_uniqueness_of :username, :email
  validates :password,  presence: {on: :create}, length: {minimum: 6}, confirmation: true
  validates :password_confirmation, presence: true, if: '!password.nil?'

  def self.find_or_create_by_omniauth(auth_hash)
    random_pass = SecureRandom.hex
    self.where(email: auth_hash["info"]["email"]).first_or_create do |user|
      user.username = auth_hash["info"]["name"]
      user.provider = auth_hash["provider"]
      user.password = random_pass
      user.password_confirmation = random_pass
    end
  end
end
```

**The Nested Nightmare**

I had quite a fair bit of trouble with nested forms being saved when the parent object was being submitted. In the case of this app a `Review` `belongs_to` a `Writer` and a `Writer` `has_many` `Reviews`. I found that, possibly as a Rails 5 development (I'll look into it) that adding `inverse_of` after the association solves this issue. See the below snippet of the `Writer` model and the first line of **Fig.1**. I also had trouble incoprorating `Carrierwave`'s uploaders into the nested form and the only solution I could find was the use `accepts_nested_attributes_for`, which is a 'no no' for this project. So I decided to remove validations for the images and when the `User` had complete the other input fields, upon submit, they would be rerouted to the edit page for that newly created `Review` and be directed to upload the required images by an alert box, this was far from ideal but it works at least. I think perhaps following my assessment I might implement the `accepts_nested_attributes_for` technique I found, or perhaps make a new template featuring fields only for the images and have the `User` be routed to that.

```
class Writer < ApplicationRecord
  has_many :reviews, inverse_of: :writer, dependent: :destroy

  validates_uniqueness_of :name
  validates_presence_of :name, :publication, :bio
  validates :name, :publication, length: { in: 3..25 }
  validates :bio, length: { in: 100..3000 }

  def lastest_review
    self.reviews.order("created_at DESC").first
  end

  def review_list
    self.reviews.order!(:title)
  end

  def reviews_attributes=(reviews_attributes)
    reviews_attributes.each do |i, review_attributes|
      self.reviews.build(review_attributes)
    end
  end
end
```

**Controlling the Home Page**

The only controller I'll go into in depth is the `HomeController`, the others have a fairly standard set up and are quite similar to one another and the `CommentsController` is covered in the link provided above. The index action's job was to fetch information from the `Review` model and populate the carousel feature within the home page's HTML (essentially the main item that the `User` encounters as a 'landing page' - in fact the only other stuff the `User` sees at this point is the `_header` and `_footer`). I had the index action perform the basic function of brining all the `Reviews` to one place and then letting the model do the rest.

**Fig.5**
*./app/controllers/home_controller.rb*

```
class HomeController < ApplicationController
  def index
    @reviews = Review.all
  end

  def admin_area
    @reviews = Review.order(:title)
    @writers = Writer.order(:name)
    @genres = Genre.order(:genre_name)
  end
end
```

The within the home page's carousel, I called `@reviews.latest_review` (or `@reviews.second_latest_review` and so on, respectively, for the last four `Reviews` uploaded) and then chained on the attributes I wanted to display such as `title` or `writer.name`. This would mean that my client could upload the latest review via the app's **admin role** user interface and expect to see the latest review be the first featured item on the carousel when a visitor navigates to the app's landing page. Referring back to **Fig.1**, inside the model, below my uploaders, I wrote four class scope methods with the above titles and used `offset`, then`limit` and first to capture and isolate the particlaur `Review` I wished for the model to return. This was a long project and even though I had other reasons for it taking some time, I do feel that I spent perhaps too much time dwelling on details and appear - all while I still have Javascript ahead of me. That said, it was a great deal of fun, I learnt a ton and I certainly got a laugh out of these scathing critiques!
