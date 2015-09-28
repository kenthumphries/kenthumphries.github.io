---
layout: post
title: NSDates, NSDateFormatters, NSTimeZones, NSCalendars and NSDateComponents. Oh my.
---

Summary
-------------

0) From WWDC2013, watch [Solutions to Common Date and Time Challenges](https://developer.apple.com/videos/wwdc/2013/).

1) Date AND time? Use NSDate.

2) Time only? Use NSInteger.

3) Timezone? Store name as NSString.

4) Date only? Use NSDateComponents.

5) 6) 7) Comparing minutes/hours/days/weeks? Use  [NSCalendar components:fromDate:toDate:options](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSCalendar_Class/#//apple_ref/occ/instm/NSCalendar/components:fromDate:toDate:options:)




1) What's the best option to store a time on a specific date?
-----------------------------------------------------------------------------------

This is the primary (only?) case where NSDate should be used. The name of this class can be misleading. It's not just a date, it's a universal time, the seconds since the [new millennium began](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSDate_Class/#//apple_ref/occ/instp/NSDate/timeIntervalSinceReferenceDate). i.e. It's a date AND a time. It's an instant in time, although it doesn't take timezone into account.

If you want to store a time on a certain day, this is your best bet.

2) What if I only want to store a time?
--------------------------------------------------

A simple NSInteger is a good bet. Determine the seconds since midnight:
    seconds + (60 * (minutes + (60 * hours)))

3) How do I store a time (or NSDate) in a certain timezone?
------------
Best to keep the two things separate, then put them together before you do any relevant calculations (NSCalendar), or display to user (NSDateFormatter). You'll need to create a tuple with your NSInteger or NSDate and the timezone as an NSString. Storing the timezone name (i.e. "Australia/Melbourne") as an NSString makes it [easily human identifiable](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) and easy to [initialise into an NSTimeZone](https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Reference/Foundation/Classes/NSTimeZone_Class/index.html#//apple_ref/occ/clm/NSTimeZone/timeZoneWithName:).

4) What if I only want to store a date?
--------------------------------------------------

*Do not use NSDate, please. I beg you.*

Use NSDateComponents and fill the components that are relevant. If you want a day of the week, fill only the [weekday](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSDateComponents_Class/#//apple_ref/occ/instp/NSDateComponents/weekday) component. If you want a day on a certain year, fill [year](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSDateComponents_Class/#//apple_ref/occ/instp/NSDateComponents/year), [month](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSDateComponents_Class/#//apple_ref/occ/instp/NSDateComponents/month), [day](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSDateComponents_Class/#//apple_ref/occ/instp/NSDateComponents/day)!

It might seem a bit unwieldy, but it will prevent confusion on everyone's (that includes *you*, future me!!) part. You're not giving extra information to the date that can be misused and potentially abused.

5) How do I compare two dates?
-------------------------------------------
    [NSDate compare:]

6) Smartass. I meant, how do I figure out the days between two dates??
----------------------------------------------------------------------------------------------------
  
So you want to know how many days are between the 23rd of June and the 7th of July? Well, [thirty days hath September, April, June](https://en.wikipedia.org/wiki/Thirty_days_hath_September)... Just kidding. But you do need to keep in mind that months have different numbers of days and 1 actually comes after 30, or 31, or 28....or sometimes 29. Just use [NSCalendar components:fromDate:toDate:options](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSCalendar_Class/#//apple_ref/occ/instm/NSCalendar/components:fromDate:toDate:options:) and let Apple do the maths for you.

7) How do I figure out the days between two dates, taking timezone into account?
--------------------------

If you're talking timezones, that means you have a time and a timezone. If you're talking days, that means you have a date. What's the best option to store a time on a specific date? How do I store a time (or NSDate) in a certain timezone? All that stuff you learnt before *is* useful! Yes, NSDate and an NSString containing the timezone name.

You can create an NSTimeZone using your timezone name, and from that an NSCalendar aligned to your timezone. Then use good old [NSCalendar components:fromDate:toDate:options](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSCalendar_Class/#//apple_ref/occ/instm/NSCalendar/components:fromDate:toDate:options:).

However, this will assume that 'day' means 24 hours apart. What if you mean "how many sleeps between now and my flight to Toulouse?" and your flight is 8am in the morning, and right now it's 8pm 2 days before? You want to know it's 2 sleeps, not 36 hours == 1 full day. 

Modified from [this SO post](http://stackoverflow.com/a/28921565):

    -(NSInteger)daysFromDate:(NSDate *)lhs toDate:(NSDate *)rhs inTimeZone:(NSString *)timeZoneName
    {
        NSDate *lhsStart;
        NSDate *rhsStart;

        NSTimeZone *timeZone = [NSTimeZone timeZoneWithName:timeZoneName];
        NSCalendar *cal = [NSCalendar calendarWithIdentifier:NSCalendarIdentifierGregorian];
        cal.timeZone = timeZone;

        // These lines are key and ensure days only are taken into account, not hours of the day
        [cal rangeOfUnit:NSCalendarUnitDay startDate:&lhsStart interval:NULL forDate:lhs];
        [cal rangeOfUnit:NSCalendarUnitDay startDate:&rhsStart interval:NULL forDate:rhs];

        NSDateComponents *difference = [cal components:NSDayCalendarUnit fromDate:lhsStart toDate:rhsStart options:NULL];
    
        return difference.day;
    }


**This assumes both dates are in the same timezone. Comparing dates in different timezones seems [tricker](http://stackoverflow.com/a/2066255).** 
