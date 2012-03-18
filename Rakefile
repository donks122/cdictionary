
require 'rubygems'
require 'rake'
require 'nokogiri'

task :default => 'build:html'


class LCHelper

  CONTENT_FILE = 'commands.xml'

  class Node

    attr_accessor :children, :item

    def initialize( item ) 
      @item = item
      @children = [ ] 
    end

    def add_child( child )
      @children << child
    end
  end

  class << self

    def doc 
      @doc = Nokogiri::Slop( 
        File.open( CONTENT_FILE  ).read
      ) unless @doc

      @doc
    end

    def category_nav_for node_list 
      if node_list.size > 0
        markup = '<ul>'
        node_list.each do |i|

          next if i.item[ 'name'].nil?

          markup << "<li> <a href='\##{i.item[ 'name' ]}'>" << i.item[ 'display' ] << "</a>"
          markup << category_nav_for( i.children ) if i.children.size > 0
          markup << '</li>'
        end
        markup << '</ul>'
      end
    end

    def category_markup
      category_nav_for @node.children
    end

    def parse_rest node_list
      if node_list.length > 0 
        node_list.each do |cat|
          item = Node.new( cat )
          @current_node.add_child( item ) 
          if cat.css( 'category' ).length > 0
            tmp = @current_node
            @current_node = item 
            parse_rest cat.css( 'category' )
            @current_node = tmp
          end
        end
      end
    end

    def parse 
      @current_node = @node = Node.new( self.doc.css( 'root' ).first )
      parse_rest( self.doc.css( 'root > category' ) )
      @node
    end

    def _r fname, dir = 'html'
      f = File.join( 'assets', ( dir || 'html' ), fname ) 

      File.open( f ).read  if File.exists? f
    end

    def get_header
      _r( 'head.html' )         << 
      _r( 'style.css', 'css' )  << 
      _r( 'app.js', 'js' )      <<
      _r( 'top.html' )
    end

    def get_footer 
      _r( 'foot.html' )
    end

  end
end



namespace 'build' do

  task :html do
    LCHelper.parse
    File.open( 'build.html', 'w' ) do |f|
      f.write LCHelper.get_header
      f.write LCHelper.category_markup
      f.write LCHelper.get_footer
    end
  end

end



