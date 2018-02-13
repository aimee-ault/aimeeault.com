title: What the Hell Should I Stub in My Tests?
link: http://aimeeault.com/2016/04/13/what-the-hell-should-i-stub-in-my-tests/
author: aimeeault
description: 
post_id: 1752
created: 2016/04/13 02:52:21
created_gmt: 2016/04/13 02:52:21
comment_status: open
post_name: what-the-hell-should-i-stub-in-my-tests
status: publish
post_type: post

# What the Hell Should I Stub in My Tests?

Until very recently, one of my job responsibilities at [Treehouse](http://teamtreehouse.com) was serving the role of a mentor to a junior developer. It was a role that I appreciated because it sometimes called to question my own understanding of things that I have always assumed I knew as someone with over a decade of experience. And sometimes I know things without really knowing why I know those things, and that makes a spectacular challenge in helping educate someone who is eager to learn--to the extent that I sometimes had to relearn things I already know, this time with more intellectual foundation. Another of my job responsibilities is attempting to make significant improvements to our codebase at a higher level. Not long ago, these two things collided when I decided to try my hand at cleaning up our test suite, which can take upwards of 15 minutes (with parallel pipelines!) to run in CodeShip. That's a far cry from where we were a few months ago, when our test suite was finishing builds in a speedy 3-5 minutes. Beyond feature tests, which are meant to be literal end-to-end tests, I noticed that it's not always obvious to developers when you should stub objects or calls to methods in unit tests. And as I had a discussion with my former mentee, Monique, about it, I realized it's sometimes not even clear to me what my principles are on the subject sometimes. What better a way to document your principles than by deconstructing them? Here's some of my principles, that you might be able to relate to. 

### Unit tests should adhere to single responsibility patterns

If your class adheres to single responsibility patterns, why _wouldn't_ your tests for it? This should be a no-brainer.  Pretend you have a service object called `SignupUser`. Its purpose is mostly clear in its name and it might be obvious to you that it's going to use instances of other classes in fulfilling its purpose, namely `User`. This is contrived, but bear with me. 
    
    
    class SignupUser
      attr_reader :name, :email
    
      def initialize(name, email)
        @name = name
        @email = email
      end
    
      def execute
        user = User.new(name: name)
        if user.save
          Logger.build_event(user, "New Signup")
          UserMailer.welcome_mailer(user).deliver_now!
        end
      end
    end
    

This is such a simple class but you can see that this method is creating a `User` object and subsequently making calls to a mailer and an event-logging system. As the app evolves, it may grow as well, but generally the things it does are limited to this particular interaction. Who knows what's happening in the event logger or mailer? You shouldn't need an awareness of how other class's methods work when you're building a unit test. The unit test should test the behavior of the class being described and so it is fair game to _not_ extend your unit tests to the behaviors of those classes. The methods for those classes should be tested within their own respective unit test class. Another old favorite is when a controller calls a method on a model and the controller is tested against behavior that took place inside of the model: 
    
    
    class ItemsController
      def archive
        item = Item.find(params[:id])
        item.archive!
        head :ok
      end
    end
    
    
    
    class Item < ActiveRecord::Base
      def archive!
        update_attribute(:archived_at, Time.now)
      end
    end
    
    
    
    describe ItemsController do
      describe "PUT 'archive'" do
        let(:item) { create(:item) }
        let(:params) { id: item.id }
    
        it "archives the item" do
          put :archive, params
          expect { item.reload.archived_at }.to_not be_nil
         end
      end
    end
    

This is testing behavior dictated by `Item`. Simple behavior, yes, but you could just set an expectation that the method was called, which stubs it in the process: 
    
    
    describe ItemsController do
      describe "PUT 'archive'" do
        let(:item) { create(:item) }
        let(:params) { id: item.id }
    
        it "archives the item" do
          expect(item).to receive(:archive!)
          put :archive, params
        end
      end
    end
    

Let's be clear here, though: This approach to testing walks a really fine line when it comes to ensuring full test coverage. I showed this example to my boyfriend, also a developer, and we almost broke up over it. The police even came out to our apartment to break up the ensuing fisticuffs. I mean not really, but what I'm getting at is this: you may have your own opinions here. And that's cool. As long as you know why you have that opinion and that you've decided that what you write is the best fit for your test! Capisce? 

### Don't test the framework you're using.

Occasionally, I come across unit tests for ActiveRecord models that test against the behavior of Rails and ActiveRecord. This isn't necessary and it can actually lag your test suite's runtime because it is guaranteed to produce database transactions. For example, you shouldn't need to test for persistence on creation of an ActiveRecord model. If you have a valid model, it's implicit that it will properly be saved. However, behaviorally, it's fair game to test your validations. 

#### Bad test:
    
    
    user = User.create(email: "foo@bar.com", name: "Bob") 
    expect(user.persisted?).to be_truthy
    

#### Good test:
    
    
    user = User.new(email: "foo@bar.com", name: nil) 
    expect(user).to_not be_valid
    

In a similar fashion, it's also not uncommon to see record persistence being tested within a controller test: 
    
    
    describe ItemsController do
      describe "POST 'create'" do
        let(:params) { name: "Foo", description: "Bar" }
    
        it "creates an Item" do
          expect { post :create, params }.to change { Item.count }.by(1)
        end
      end
    end
    

![anigif_optimized-12950-1454940032-1](/wp-content/uploads/2016/04/anigif_optimized-12950-1454940032-1.gif) This is completely unnecessary and testing that the controller makes a call to save should suffice: 
    
    
    it "creates an item" do
      expect_any_instance_of(Item).to receive(:save!).and_return(true)
    end
    

### Don’t Test Against APIs

This came up during a conversation with a friend of mine. He was working on a side project developed in Rails and was a bit green with test-writing and was asking for my opinions on a few things and somehow it came up that he wasn’t mocking requests to an API he was using. This wasn’t for any “dumb” reason—he just wasn’t sure how you were supposed to account, in your own application, for changes in response objects from third party APIs. And well: you shouldn’t. That’s not on you. But I can understand that that is a hard answer to swallow (the real answer, in my opinion, is that any API that changes so drastically that it breaks your app without bumping version number is a really crappy API and you should consider not using it). 

#### Disable everything by default.

I appreciate gems like [Webmock](https://github.com/bblimke/webmock), which allow you to universally prevent all outside requests in your test suite. Not just because they can lead to slow test suites and oh-so-joyous timeout-related failures, but because well, some APIs really suck. Recently in my job, I was reviewing a pull request for someone and noticed something curious. He had added an initializer which disabled all outbound requests to this API in development or staging environments. I immediately questioned that. “The API doesn’t have a sandbox, just production.” And I grimaced. But imagine a world where Webmock didn’t exist and someone forgot to stub a request and suddenly there it is: test data in your production environment. Gross. What a nightmare. 

#### Spec helpers can be your friend.