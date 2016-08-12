# Calculate Upgrade Cost

Honestly the trickiest part of upgrading or downgrading somebody's account is you want to make sure that you tell accurately what they will be charged to make the change. You don't want to mess that up and then charge something different, so here's what we're going to do. As soon as this screen loads here, we're going to make an ajax call back to the server. On the server we're going to calculate how much the User would be charged if they upgraded from the Farmer Brown Plan to the New Zealander Plan.

Take into account that they may already be partially through a paid month. First, in profile controller, let's make a new public function called preview plan change action. URL will be something like slash profile, slash plan, slash change, slash preview, slash plan, ID. I'll give it a name of Account Preview Plan Change, so this end point won't actually make the change it will just try to guess what that change would look like.

Since we're being passed the plan ID that we're going to change to, we'll go ahead and fetch the plan object by getting the subscription helper and calling fine plan on that. The big question is, how the heck do we figure out how much the Users going to be charge? Well fortunately, Stripe takes care of this for us and the killer feature here, if you look at the API reference and the invoices is something called In Upcoming Invoice.

The idea is this, you can ask Stripe what the Users next invoice would look like. This could be used for a couple a different things. This can be used to figure out how much the User will be charged on their next cycle of renewal, or you can use it to ask it how much the User would be charged if you passed it a different plan? Specifically here if we passed the subscription underscore plan, with the new plan ID then Stripe will figure out how much they should be charged for switching that plan.

Now keep part of this as the subscription pro rate, and if you look at the subscription upgrading and downgrading part they talk about this pro rating stuff. In fact they have a lot of text here talking about what exactly happens in different scenario's. By fault Stripe does pro rate the cost, which means that if we are half way through the month on the Farmer Brown Plan and we upgrade then we should be credited for already having paid for the second half, so we'll be discounted on what we paid for the second half.

Long story short the compass calculations are taken care of for us, all we need to do is ask Stripes API for the upcoming invoice if we use this new plan. Let's make a new function inside of our Stripe client that uses the API to get the upcoming invoice, we call that Public Function Get Upcoming Invoice For Changed Subscription. This is taking two arguments the User so we can figure out which customer we're going to be upgrading, and also the subscription plan object which will be the new plan that we want to change to.

Very simply we just going to return slash Stripe, slash invoice, colon, colon, upcoming, and all passes a couple of a parameters here. The required parameters are customer, because we need to know which customer we're going to be invoicing, second we need to know the subscription. In other words which subscription is changing, even though in our system a customer will always only have one subscription it's nice to pass this. The last one is subscription underscore plan. This is the important one because it tells it, which subscription plan we want to change to, so if it's a new plan arrow, get plan ID and that is it.

On profile controller let's use this to get a new Stripe invoice object. Say this arrow get Stripe underscore client, arrow, get upcoming invoice for change subscription. The two arguments are the User, so this get User and the new subscription plan which is plan. Let's actually see what this looks like, so let's dump plan here and then we'll just return the new [Jason 00:06:04] response object. Pass it a total key just set to 50 for now, so we'll change that to make the real thing but we can go ahead and try to read that inside of our job script.

Off the dump here, let's dump Stripe invoice not planned, that will be a little bit more interesting. All right on the job de-script side on things let's make it so that once we click to change the plan we make an ajax called this New End Point. What we'll do here is find where our button is, way down here, and I'm going to add another [attribute 00:06:49] here called Data dash Preview URL. How we use twigs path function to link to our account preview plan change and we'll pass it Plan ID set to other plan at Plan ID.

This is called, because we should be able to use that in job de-script up here, so up top we'll say marked premium URL equals this dot data preview dash URL. Let's also while we're here get far plan name equals this dot data dash plan name which is another data attribute set, and finally let's make the ajax call. The URL is the preview URL and then on success will register a call back function, and for now we'll use the sweet alert to just say total [dot 00:08:07] sign plus data dot total to read our ajax unpoint.

Kind of a functional front end, and then the back end we're going to start dumping to get some debug information, so let's try it out. Refresh the page, I'd change to New Zealander, total 50 dollars which we have hard coded back from our [inaudible 00:08:26] and down here because I'm using symphony it actually recorded that ajax call. If I click to go onto the profiler for it I can scroll down here, go to debug and this is actually the dumped value we have.

If we not using symphony, you'll need to dump that value in a different way. Now this object looks a little bit ugly but when we're looking here is for this values section, and there's a couple of interesting things. First of all there is an amount underscore, do, which is what we're going to actually use and share with the User, and remember everything is done in cents. We're going to want to divide this by 100 before we show it to the User.

Another interesting thing is it shows you the invoice line items, and whenever you change a subscription there's always going to be three invoices line items. The first one as you'll see is the unused time on the existing plan, it will actually give you a negative number as a credit. The second one is actually the remaining time, so whatever the rest of your month is, on the new plan. When you upgrade plans it doesn't change your billing cycle it just switches you in the middle of the billing cycle. Now the third one is a little bit weird, the third one is actually the full amount for the new plan.

This is the tricky part, when you upgrade with Stripe, by default if I upgrade right now Stripe doesn't bill me right now. Instead it allows me to switch to the New Zealander, and then at the end of the month when I am normally billed for a renewal, it will bill me for the cost of the New Zealander plan plus the pro rated cost for the time that I just used. That's why you have these first two line items which are kind of the together the total we'd expect, plus the cost for the full price renewal.

Now this is a little bit weird because usually what you want is when you upgrade I want to charge them right now, and I want to charge them whatever the pro rated difference is for upgrading from one plan to another in the middle of the month. That is actually what we're going to do, we're going to charge the User right now for their change. What this means is when we read this amount due value off of here we're going to need to subtract the price of the plan to remove this cost here.

Actually we're going to bill them just for the negative and positive pro rations that they should owe if we billed them right now. In the profile controller what this means is that we'll start with total equals, Stripe invoice arrow amount underscore due. Nice and simple. I'll make a note here that this includes the pro rations plus next cycles full due amount. To get the real total for us we're going to say total minus equal nil set plan arrow get price, but times 100 because this plan object is our nice subscription plan object.

We store the amount, the price there in dollars, so we convert that to cents so that we get the actual total in cents. Then finally down here when you change 50 to total divided by 100, so we return that to the front end in dollars so it shows nice to the User. Let's go back refresh, hit change, and that looks correct, because I'll remind you that the difference between these two plans is about 100 dollars. Since we just signed up a few minutes ago it makes sense that the price here is just a little bit under a 100 dollars.

If we were halfway through the month then it would be even cheaper. Now before we actually, last up here is actually to exit the change. To do that I'm going to copy in a little of job de-script to save us some time and that's down here inside of a done. I'm going to paste in some job de-script that does a couple of things, first it's going to show them a really nice message that they know exactly how much they're going to be charged. Now notice the amount they might be charged could be positive in which case they'll billed that immediately or it could be negative if you're downgrading a plan. In that case the User and Stripe is actually going to get a balance that will be automatically applied to future invoices. That's the message that we give them there.

Finally I show them one last alert that allows them to confirm that yes, this is actually what I want to do. If they click, Okay, in a second we're going make one last ajax call to actually change their subscription and bill them. Let's just see how this looks, beautiful, so let's hook up this, okay, button to make this ajax call and change their plan.