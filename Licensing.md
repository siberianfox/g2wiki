## Intent
The g2core **project** is a motion control **application** built on a set of underlying **components**. We want the licensing to reflect this, and to treat using the application as a whole somewhat differently than using the components as a library. The intent of the g2core licensing scheme is summarized as:
* Ensure that the g2core project remains open source
* Encourage contribution to the project and ensure that most changes and enhancements are returned to the community
* Make it easy to use g2core components in free and commercial projects/products- i.e. encourage use of g2core components as a library
* Notwithstanding the above, make the application when used as a whole retain GPLv2 copyleft and other provisions

G2core licensing is based on GPLv2 with the [BeRTOS extension](http://www.bertos.org/discover/license) to enable using component files without invoking GPL copyleft. To this end, all files in the project are licensed under GPLv2. Those files that are considered "components" carry the BeRTOS exception, that allows their use without opening source code that they come in contact with.

G2core also includes [Motate](https://github.com/synthetos/Motate) hardware abstraction components that are licensed under the same scheme.

Most of what follows is shamelessly cribbed from the BeRTOS site.

## Info
### You are free to...
* Include g2core components within any product, distributed under any license, including commercial licenses and/or closed-source licenses
* Modify g2core components as you want under the following conditions:
 * Attribution - You must declare in a written statement in documentation and source code that you are using g2core components in your application and offer to provide the (possibly modified) g2core source code
 * Share-alike - If you modify g2core components, you may distribute them only under the original license
 * Contribution - If you modify g2core components, you must contribute the modifications back to the g2core project

### What you can do with g2core components...
If you are a company or individual doing commercial embedded products, you can:
* Download and use g2core components as you want
* Sell products based on g2core components without paying royalties or other fees
* Sell products based on g2core components without opening or giving away your other application source code

### What you must do with g2core components...
If you sell or otherwise distribute code/products that use g2core components you must:
* Provide attribution that you are using g2core components in the documentation and source code. Text like this is sufficient:
<pre>
"This product uses g2core components (http://www.synthetos.com), Copyright 2013 - 2016"
</pre>
* If you modified g2core components offer the modified versions for download from your website. Alternately, you can contribute your modifications to the g2core project, where they may be integrated in whole or in part, and/or hosted independently under the project if not integrated.

## Text
<pre>
/*
 * g2core.h - g2core main header - Application GLOBALS
 * Part of g2core project
 *
 * Copyright (c) 2013 - 2106 Alden S. Hart Jr.
 * Copyright (c) 2013 - 2016 Robert Giseburt
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
