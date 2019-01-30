---
title: "Re-purposing NUnit Test Framework for hardware testing"
layout: post
date: 2018-01-29 12:00
tag: 
- C#
- NUnit
- Testing
headerImage: false
projects: true
hidden: true # don't count this post in blog pagination
description: "Testing Solid State Power Controllers (SSPCs) using NUnit unit testing framework"
category: project
author: Jerry Li
externalLink: false
--- 

## Overview
Testing software is important. It is especially so in the aerospace industry. One of the projects I worked on was testing the software of a Solid State Power Controller (SSPC) for an aircraft. Think of the SSPC as a glorified circuit breaker, you put power into the card, and distribute it to other systems by turning channels on and off /(controlled with software/). A SSPC is a piece of hardware running software so testing the software is quite difficult. We could approach it one of two ways: we could solely isolate the software and unit test the software by emulating the microcontroller running the software or we could test the software at a system level, we interface with the card at a hardware level and verify the hardware behaves how we expect it to. For this project, I was tasked to test the software at a system level - all interfacing with the software was done through the communications port. The interesting part of this testing project is the melding of software control of test equipment, unit testing paradigms, and SSPC communications to test the software on the SSPC.

Before going further, I should give a brief overview of the unit under test. As mentioned earlier, a SSPC can be thought of as a glorified circuit breaker. In a SSPC, there are input channels that connects to a power supply and multiple output channels that connect to other systems that require power. The On/Off functionality of the SSPC is controlled over a Controller Area Network (CAN) bus. Commands can be sent over the CAN bus to read the voltage, turn on or off channels, set the current trip levels, and read the status of the card. At a high level, to test the SSPC, we connect the inputs to a test power supply and the outputs to an electronic load. We send commands to turn on and off channels and draw loads from the channels and verify the behavior corresponds to what we expect. At a more detailed level, a test fixture was developed for the SSPC. The test fixture has added connections for an oscilloscope and digital multimeter to measure waveforms and voltage levels. A matrix switch card was used to multiplex the test equipment to the SSPC connections to allow for fine-grain control of signal measurement.

The rest of the post will focus on the software application designed for the testing of the SSPC.

## Design
The hardware test environment consists of 2 parts that require software: the test equipment and the CAN communications adapter. With that knowledge, the test application will be comprised of three parts: the libraries necessary to control the test equipment, the libraries necessary to control the CAN communications adapter, and the testing framework. 

### Test Equipment Control
The de facto method of controlling test equipment is using IVI drivers. Most equipment support control using the C programming language or a language that supports the Component Object Model (COM) such as C#. C# was used for this project because of the high-level language features made available to the developer. The decision to use C# was the main motivator to choose NUnit as the testing framework. 

### CAN communications adapter
Any CAN adapter could be used as long as it supports the CAN protocol. In this case, a Peak PCAN CAN-to-USB adapter was used because I had access to an existing C# library for the adapter. There was also an existing command library for SSPC written for the PCAN adapter.   

### Unit Test Framework
As stated in the title, NUnit is used as the test framework. This was chosen because C# was chosen as the language to control the test equipment. In essence, the test framework was chosen around the other two parts of the test environment.

## Implementation
To test the SSPC, the basic execution goes through five steps:

**Initialization -> Test Equipment Manipulation/SSPC Control via CAN Commands -> Read status from SSPC via CAN Commands/measure values using test equipment -> Assert measurements/status are within expected values -> Teardown** 

The testing of the SSPC is not too different than unit testing other software. For this project, the goal was to break the SSPC test cases into basic "test units" and divide the tests so that each test only verifies one basic function of the SSPC. Once we do that, we can see that NUnit fits nicely as the framework to structure the tests. 

The tests were divided into multiple broad categories /(such as voltage tests, communications tests, etc/). Each of these categories were logically grouped into individual namespaces so that NUnit can easily bucketize the tests. The tests themselves had multiple steps and multiple "verify" statements, each test /(which we called procedure/) was a "TestFixture" under NUnit. The procedure was broken up into logical fragments and each fragment is a NUnit "Test". A fragment may be something as small as "turn on the channel and verify the multimeter measures voltage at the output." Following the unit testing paradigm, I tried keeping all the fragments as small as possible and also divided the fragment so that it only tested one functionality.

Since the test equipment and CAN adapter initialization are the same throughout the tests, it only needs to be in one place. This was a good use of the "OneTimeSetUp" attribute, it's called once at the beginning and won't be called again until the test is re-executed. Wait, this putting the initialization in the "OneTimeSetUp" would only affect a fixture - this can be easily fixed by putting the actual setup in a base class and extending all tests based on that. Just because there's a "OneTimeSetUp" doesn't negate the need for a test-specific setup and teardown. While it's possible that a test may need a setup or teardown, not every test needs it since the instrument init is handled elsewhere.

Earlier, it was mentioned that each test was a fragment of the procedure. Each fragment/test may be run multiple times with different inputs. The inputs were generated from a separate class. This made the layout quite straightforward. One section contained the test logic, another contained test-specific setup and teardown, another section was for any private helper functions specific to the procedure, and finally another section that contained the test vectors.

Eventually the file structure was as follows:
* Base Class 1 in Namespace 1
    * Procedure 1
	* Procedure 2
	* ...
* Base Class 2 in Namespace 2
    * Procedure 1
	* Procedure 2
	* ...
* ...

The base class was quite simple and formatted in the following manner:
```csharp

namespace namespace1
{
	[SetUpFixture]
	class BaseClassSetupFixture
	{
		[OneTimeSetUp]
		public void OneTimeSetUp()
		{
			// Initialize instruments and communications modules
		}
		[OneTimeTearDown]
		public void OneTimeTearDown()
		{
			// Close instruments and communications modules
		}
	}
}

```

Each procedure followed the following format:
```csharp

namespace namespace1
{
	[TestFixture]
	class Procedure
	{
		[SetUp]
		public void OneTimeSetUp()
		{
			// Test specific setup
		}
		[TearDown]
		public void OneTimeTearDown()
		{
			// Test specific teardown
		}
		
		[Test]
		[Order(1)]
		[TestCaseSource(typeof(ProcedureTestVectors), nameof(ProcedureTestVectors.testVectors))
		public void TestFragment1(Object input1, Object input2)
		{
			//Test fragment logic
		}
		[Test]
		[Order(2)]
		[TestCaseSource(typeof(ProcedureTestVectors), nameof(ProcedureTestVectors.testVectors))
		public void TestFragment2(Object input1, Object input2)
		{
			//Test fragment logic
		}
		...
	}
	
	class TestFragment1TestVectors
	{
		public IEnumerable<Object[]> testVectors()
		{
			//logic to generate input vectors
		}
	}
	class TestFragment2TestVectors
	{
		public IEnumerable<Object[]> testVectors()
		{
			//logic to generate input vectors
		}
	}
}

```
Obviously, in the actual test application, the names of the variables, classes, and tests were named in a logical manner that corresponding to its use.

## Final Thoughts
* There is a lot more information that wasn't included in this post, this whole project was a cumulation of over a year's worth of research.
* As with most re-purposing efforts, not everything works nicely together - in this case, the Unit Testing paradigm doesn't allow user input during test execution /(and that makes sense, you should use a mock for that purpose/) but when we're testing hardware, sometimes human intervention is necessary and NUnit doesn't provide a good way for user input.
* Sometimes, testing requires long wait periods for equipment to reset and initialize. The code has many Sleep statements that just look ugly and feels like a code smell.

**The test equipment automation was achieved with IVI drivers. Learn more about them [here](http://www.ivifoundation.org/).**

**The unit test framework used was NUnit. Learn more about NUnit [here](https://github.com/nunit/nunit).**
