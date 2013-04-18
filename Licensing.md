## Intent
The tinyg2 **project** is a motion control **application** built on a set of underlying **components**. We want the licensing to reflect this, and to treat using the application as a whole somewhat differently than using the components as a library. The intent of the tinyg2 licensing scheme is summarized as:
* Ensure that the tinyg2 project remains open source
* Encourage contribution to the project and ensure that most changes and enhancements are returned to the community
* Make it easy to use tinyg2 components in free and commercial projects/products- i.e. encourage use of tinyg2 components as a library
* Notwithstanding the above, make the application when used as a whole retain GPLv2 copyleft and other provisions 

tinyg2 licensing is based on GPLv2 with the [BeRTOS extension](http://www.bertos.org/discover/license) to enable using component files without invoking GPL copyleft. To this end, all files in the project are licensed under GPLv2. Those files that are considered "components" carry the BeRTOS exception, that allows their use without opening source code that they come in contact with. 

Most of what follows is shamelessly cribbed from the BeRTOS site.

## Info
###You are free to...
* Include tinyg2 components within any product, distributed under any license, including commercial licenses and/or closed-source licenses
* Modify tinyg2 components as you want under the following conditions:
 * Attribution - You must declare in a written statement in documentation and source code that you are using tinyg2 components in your application and offer to provide the (possibly modified) tinyg2 source code
 * Share-alike - If you modify tinyg2 components, you may distribute them only under the original license
 * Contribution - If you modify tinyg2 components, you must contribute the modifications back to the tinyg2 project

###What you can do with tinyg2 components...
If you are a company or individual doing commercial embedded products, you can:
* Download and use tinyg2 components as you want
* Sell products based on tinyg2 components without paying royalties or other fees
* Sell products based on tinyg2 components without opening or giving away your other application source code

###What you must do with tinyg2 components...
If you sell or otherwise distribute code/products that use tinyg2 components you must:
* Provide attribution that you are using tinyg2 in the documentation and source code. Text like this is sufficient:
<pre>
"This product uses tinyg2 components - http://www.synthetos.org), Copyright 2013"
</pre>
* If you modified tinyg2 components offer the modified versions for download from your website. Alternately, you can contribute your modifications to the tinyg2 project, where they may be integrated in whole or in part, and/or hosted independently under the project if not integrated.

##Text
<pre>
/*
 * tinyg2.h - tinyg2 main header - Application GLOBALS 
 * Part of TinyG2 project
 *
 * Copyright (c) 2013 Alden S. Hart Jr. 
 * Copyright (c) 2013 Robert Giseburt
 *
 * This file ("the software") is free software: you can redistribute it and/or modify 
 * it under the terms of the GNU General Public License, version 2 as published by the 
 * Free Software Foundation. You should have received a copy of the GNU General Public 
 * License, version 2 along with the software.  If not, see <http://www.gnu.org/licenses/>.
 * 
 * As a special exception, you may use this file as part of a software library without 
 * restriction. Specifically, if other files instantiate templates or use macros or
 * inline functions from this file, or you compile this file and link it with  other 
 * files to produce an executable, this file does not by itself cause the resulting 
 * executable to be covered by the GNU General Public License. This exception does not 
 * however invalidate any other reasons why the executable file might be covered by the 
 * GNU General Public License. 
 *
 * THE SOFTWARE IS DISTRIBUTED IN THE HOPE THAT IT WILL BE USEFUL, BUT WITHOUT ANY 
 * WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
 * OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT 
 * SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER 
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF 
 * OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
 */
</pre>