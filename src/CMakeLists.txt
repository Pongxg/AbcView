##-*****************************************************************************
##
## Copyright (c) 2009-2013,
##  Sony Pictures Imageworks, Inc. and
##  Industrial Light & Magic, a division of Lucasfilm Entertainment Company Ltd.
##
## All rights reserved.
##
## Redistribution and use in source and binary forms, with or without
## modification, are permitted provided that the following conditions are
## met:
## *       Redistributions of source code must retain the above copyright
## notice, this list of conditions and the following disclaimer.
## *       Redistributions in binary form must reproduce the above
## copyright notice, this list of conditions and the following disclaimer
## in the documentation and/or other materials provided with the
## distribution.
## *       Neither the name of Sony Pictures Imageworks, nor
## Industrial Light & Magic nor the names of their contributors may be used
## to endorse or promote products derived from this software without specific
## prior written permission.
##
## THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
## "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
## LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
## A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
## OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
## SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
## LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
## DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
## THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
## (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
## OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
##
##-*****************************************************************************
project (AbcViewer)
SET( H_FILES
     Transport.h
     Viewer.h )

SET( CXX_FILES
     Transport.cpp
     Viewer.cpp
     main.cpp )

SET( SOURCE_FILES ${CXX_FILES} ${H_FILES} )

SET( THIS_LIBS
	Imath
	Alembic
	freeglut
    AlembicAbcOpenGL
    ${OPENGL_LIBS}
    )
	
include_directories (
	../Imath
	../alembic/lib
	../alembic
	../freeglut/include
	${GLEW_DIR}/include/
	../
	./
	.
)

add_definitions(-DNOMINMAX -D_CRT_SECURE_NO_WARNINGS -DPLATFORM_WINDOWS)

INCLUDE_DIRECTORIES( "${ALEMBIC_SOURCE_DIR}/lib" )
ADD_EXECUTABLE( ${PROJECT_NAME}  ${SOURCE_FILES} )
TARGET_LINK_LIBRARIES( ${PROJECT_NAME}  ${THIS_LIBS} )

INSTALL( TARGETS ${PROJECT_NAME} 
         DESTINATION bin )

#ADD_PYTHON_SCRIPT_INST( SimpleAbcViewerRenderit )
