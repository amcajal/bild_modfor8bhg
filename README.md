This is a fork from the original repo: https://github.com/anthonynsimon/bild

It contains modifications (related to the RNG) so it is suitable for the 8-bit Hubble Golang application. Basically, the aim is to force the lib to use the same RNG source than the "main" program that imports it, instead of generate new sources in some of their noise-generation algorithm functions.